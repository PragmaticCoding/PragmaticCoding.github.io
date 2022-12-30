---
excerpt: Global Data
hexmap: /assets/cybertanks/hexmap.png
permalink: /javafx/cybertanks/global_data
---

![CyberTank]({{page.banner}})

# Data Shared by the Entire Application


# Game Mode

Our game is going to operate in a number of different modes:

Map Building
: At some point, we need to be able to build a map.  This will mean deciding on its size and setting up the terrain types and the features.  Eventually, we'll need to be able to save the data to our database so that we can use it to play.

Set Up
: The start of every game will be to select the units of the defenders and to place them on the map.

Play the Game
: Once the setup is complete, we can start playing the game.  Here the player will be able to move units and fire upon the CyberTank.  The CyberTank will move and fire on the units.

``` kotlin
enum class GameMode { MAP_BUILD, SETUP, PLAYING }
```

# Location

All of the `Tiles` have a row and a column even though the rows are zigzag.  We're not going to try to store our `Tiles` or `Tile` related data into a two dimensional `Array` because dealing with that eventually becomes a huge pain for handling groups of `Tiles` at a time.  You end up with nested loops all over the place and it's messy.

Instead, we'll store the location of the `Tile` in the `Tile` data, and then store the `Tile` data in a simple `List`, We can use the `find()` method in Kotlin to locate a `Tile` or groups of `Tiles`.  It'll work just a well, and makes processing easier.

So we need a data structure for the `Tile` locations:

``` kotlin
data class Location(val column: Int, val row: Int) {
   fun equals(otherLocation: Location?): Boolean = otherLocation?.let {
      (this.row == otherLocation.row) && (this.column == otherLocation.column)
   } ?: false

   override fun toString(): String {
      return "$column:$row"
   }
}
```

This class will grow, for sure.  We'll probably need functions to calculate distances, and neighbours by direction and other things like that.  For now, we just need `equals()` and `toString()`.


# Game Model

To start with, our `GameModel` is fairly simple and, to be honest, it probably won't get much more complicated.  Remember that the `GameModel` is a "Presentation Model", so it contains only information that's used by the GUI.  99% of the complicated stuff in our application is going to be game-play logic, and that doesn't need data included in the GUI.

``` kotlin
class GameModel {
   val hexWidth: DoubleProperty = SimpleDoubleProperty(60.0)
   val hexHeight: ObservableDoubleValue
   val tileModels: ObservableList<TileModel> = FXCollections.observableArrayList()
   val gameMode: ObjectProperty<GameMode> = SimpleObjectProperty(GameMode.SETUP)
   val setupModel = SetupModel()

   init {
      hexHeight = Bindings.createDoubleBinding({ sin(Math.PI / 3) * hexWidth.value }, hexWidth)
   }

   fun consumeTile(location: Location, tileConsumer: Consumer<TileModel>) {
      tileModels.find { tileModel -> tileModel.locationProperty.value.equals(location) }?.run {
         tileConsumer.accept(this)
      }
   }
}
```

The only thing that's a bit tricky here is the `consumeTile()` function.  It allows the calling method to pass a code snippet which will act upon a single `Tile`.  The caller just needs to specify the `Location` of the `Tile`, and the `GameModel` will find it and run the code snippet on it.
