---
excerpt: Our first look at overall game structure and how the other parts are contained within it.
permalink: /javafx/cybertanks/game_wrapper
---

![CyberTank]({{page.banner}})

# The Game Screen

Our game screen is bascially a `BorderPane` with the Hex Map in the centre region.  The right region will hold layouts specific to each game mode.  The bottom region will be for some additional controls.

``` kotlin
class GameViewBuilder(private val model: GameModel,
                      private val paintModeWindow: Region,
                      private val tileClickHandler: Consumer<TileModel>,
                      private val animator: Runnable) : Builder<Region> {

   override fun build(): Region {
      val results = BorderPane()
      results.stylesheets.add(Objects.requireNonNull(Main::class.java.getResource("/css/cybertank.css")).toString())
      results.center = HexMapViewBuilder(model, tileClickHandler).build()
      results.right = paintModeWindow
      results.bottom = smallerButton()
      return results
   }

   private fun smallerButton(): Region = Button("Run Animation").apply {
      onAction = EventHandler { animator.run() }
   }
}
```

That's pretty small, and I don't expect it'll get much bigger than that.  The `animator` stuff is just some test code I had written to check logic for moving a cybertank around.  I've left it in because we'll probably need something like it for real game play.

# The Main Game Controller

The main Controller for the game, as you would expect, pulls everything together.

``` kotlin
class GameController {

   private val model = GameModel()
   private val interactor = GameInteractor(model)
   private val paintModeController = PaintModeController(interactor::paintTile, isVisible = model.gameMode.isEqualTo(GameMode.MAP_BUILD))
   private val setupController = SetupController { counterType, location -> interactor.toggleUnitAtLocation(counterType, location) }
   private val viewBuilder =
      GameViewBuilder(model, VBox(20.0, paintModeController.view, setupController.view, CyberTankController().view), this::handleHexClick, this::animateTank)
   val view: Region
      get() = viewBuilder.build()

   init {
     interactor.loadTiles();
   }

   private fun handleHexClick(tileModel: TileModel) {
      model.gameMode.value?.run {
         when (this) {
            GameMode.MAP_BUILD -> paintModeController.handleTileClick(tileModel)
            GameMode.SETUP -> setupController.handleTileClick(tileModel)
            GameMode.PLAYING -> TODO()
         }
      }
   }

   private fun animateTank() {
      val tankTask: Task<List<Location>> = object : Task<List<Location>>() {
         override fun call(): List<Location> {
            return interactor.calculateTankPath()
         }
      }
      tankTask.onSucceeded = EventHandler { interactor.animateTank(tankTask.value) }
      val tankThread = Thread(tankTask)
      tankThread.start()
   }
}
```

We haven't got there yet, but you can see how the Controllers for both `GameMode.MAP_BUILD` and `GameMode.SETUP` are already incorporated.  Both of these controllers implement this interface:

``` kotlin
interface TileClickHandler {

   fun handleTileClick(tileModel: TileModel)
}
```
We'll get to the `SETUP` mode controller in the next section, but we won't come back to the `MAP_BUILD` controller for a while.  It was needed in order to create the initial data set for the game, and leaving it in won't hurt anything until we come back to it.

The only other thing to note is the `init{}` section which calls `interactor.loadTiles()`.  This is going to go off to the database to get the `Tile` data, but you'll see that there's no `Task` used here.  Which you'll know, if you've read many of my other articles, is a big "no-no".  Except in Kotlin, we can use coroutines which allows us to do blocking operations on the FXAT without blocking the FXAT.  It's a wonderful feature of Kotlin which changes some of the approaches taken to operations which would ordinarily be done with `Task`.  

For now, I'm implementing the coroutine down at the DAO level, which seems appropriate to me.  I expect to revisit this as the project moves forward.

# Loading Tile Data

Now let's look at how the `Tiles` get on the screen...

First we have the `GameInteractor.loadTiles()` function:

``` kotlin
fun loadTiles() {
   model.tileModels += tileDao.readTiles().map {
      TileModel(it.location, width = model.hexWidth, height = model.hexHeight).apply {
         terrainFeatureProperty.set(it.terrain)
         terrainTypeProperty.set(it.terrainType)
      }
   }
}
```
What does this do?
First, it calls the `TileDao.readTiles()` function.  Right now this function has lots of hard-coding to pull a particular data set out of the MongoDB database.  This is a `List<TileData>`:

``` kotlin
data class TileData(val location: Location, val terrain: Int, val terrainType: TerrainType)
```

Then it transforms each of these `TileData` elements into a `TileModel` and adds it to the `ObservableList<TileModel>` in the `GameModel`.

The rest is just Reactive magic.  The Hex Map has a listener on `GameModel.tileModels` and it adds each one to the layout in its correct position.  There's no need to do anything else.  It just works.  

# The TileDao

The only other piece you need to understand how this works is the TileDao.

``` kotlin
class TileDao {

   private val client = KMongo.createClient("mongodb://127.0.0.1").coroutine
   private val database = client.getDatabase("CyberTanks")
   private val collection = database.getCollection<TileData>("Map2")

   fun readTiles(): List<TileData> {
      return runBlocking {
         return@runBlocking collection.find().toList()
      }
   }
}
```
That's all there is to it.  As previously noted, there's lots of hard-coding just to get stuff up and running without baking in a lot of process we're not ready for yet.

With the Kotlin interface to MongoDB, you just initialize a `Collection` as a certain data class, and it will de-serialize the JSON-like documents into that class when you read them.  The `find()` function with no parameters simply grabs all the data in the collection.  In this case, I had decided that a `Collection` would be a single map, to make it easy to grab the data.

The `runBlocking` command tells Kotlin to run this as a coroutine.  What this means is that it will park the process when it hits the inevitable `wait()` buried deep inside the MongoDB API while the database fetches the data.  While parked, it allows the thread - in this case the FXAT - to carry on.  When the FXAT hits another `wait()`, as it surely will when its queue is empty, then it will allow this process to carry on again if it's ready.  
