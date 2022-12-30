---
excerpt: Examining how "Drag and Drop" works and using it to deploy units on the map.
permalink: /javafx/cybertanks/dragNdrop
sidebar_image: /assets/cybertanks/setup_sidebar.png
---

![CyberTank]({{page.banner}})setup_sidebar

# Introduction

Now that the user can select how many units of each type that they want to have in the game, it's time to look at how they can deploy those units by positioning them on the game board.

# Drag And Drop

Even though drag and drop appears to be a GUI type of action, it's really about *data*.  You need to think about it as a data driven event.  For sure, it's driven by the user interacting with the GUI, and it looks like an image of something is literally being dragged across the screen, but that's all window dressing...

The Start of the Drag
: Pretty much any `Node` on the screen can be used to start a drag, if you've defined it that way.  The most important part about the defining drag for a `Node` is that you're defining the data that will be associated with the drag and you are defining the action to take place when the "drop" occurs.  

The Drop
: Once again, just about any `Node` can be defined as a source destination for the drag - the "drop" location.  The most important thing about this, though, is that you're defining the data to be transferred to the drop `EventHandler`.

With Reactive Programming
: Here's what you really need to keep in mind:  Drag and drop is a GUI driven event that allows you to deliver some *data* to an `EventHandler` which will do something with it.  That `EventHandler` has to trigger some **Game Logic** to do something with that data.  Game logic goes in the Interactor, so the job of the drop target `EventHandler`, is to pass that data, through the Controller, to the Interactor.  The Interactor will update the Model according to its game logic, and the GUI will automatically respond.

In summary, the drag takes some data from the *target* and delivers it to an `EventHandler` defined on the drag, which then invokes a `Consumer` from the Controller which will call a method in the Interactor which will (eventually) update the Model.  Because this is a Reactive system, those changes to the Model will instantly be reflected in the GUI.

But at no point are we actually moving a GUI element from one place to another!

# Drag and Drop Events

There are three drag and drop events that we are concerned about at this point:

DragDetected
: This is the `MouseEvent` that signals that the drag process has been started by the user.  It's triggered in the source `Node` via `Node.setOnDragDetected()`.  In this `EventHandler`, we'll set up the drag data and the image for the drag.

DragDropped
: This is the `DragEvent` that indicates that the user has *dropped* the image into a `Node`.  It's triggered in the target `Node` via `Node.setOnDragDropped()`.  In this `EventHandler` we'll add information to the drag data that indicates where the image was dropped.

DragDone
: This is the `DragEvent` that indicates that the drag process has been finished by the user.  It's triggered in the source `Node` via `Node.setOnDragDone()`.  In this `EventHandler` we'll pull the data from the `DragEvent` and invoke whatever logic is required to handle the processing.


# Sidebar Drag Sources

In CyberTanks!, the user will deploy the units by dragging one of the counter images onto the map and dropping it in one of the hex `Tiles`.

Now, let's look at how the drag sources are defined in the sidebar.

``` kotlin
private fun createDragableImageView(unitType: UnitType, totalProperty: ObjectProperty<Int>, deployedProperty: ObjectProperty<Int>) =
   ImageView(unitType.infoImage).apply {
      setOnDragDetected {
         if (totalProperty.value > deployedProperty.value) {
            startDragAndDrop(TransferMode.COPY).apply {
               dragView = unitType.image
               setContent(mapOf(Pair(dataFormatUnitType, unitType)))
            }
         }
         it.consume()
      }
      onDragDone = EventHandler { evt ->
         (evt.dragboard.getContent(dataFormatLocation) as Location?)?.let { deployHandler.accept(unitType, it) }
         evt.consume()
      }
      setOnMouseEntered { cursor = Cursor.HAND }
      setOnMousePressed { cursor = Cursor.MOVE }
   }
```

This is the `createDragableImageView` from the sidebar ViewBuilder.  It instantiates an `ImageView` and then sets up the drag and drop handlers for it.  

The first thing it checks is to see whether the drag is allowed.  The Model has two properties for each `UnitType`; the total number of units of that type, and the number that have been deployed.  If the total number is higher than those deployed, then there still are undeployed units, and the drag is allowed.

Isn't that business logic?
: Possibly.  Here's the way I think of it:  The model has both pieces of data clearly identified, and checking to see if one is greater than the other is simply using that data in a basic way in the GUI.  The guideline I use is that any simple combination of Model data elements is fair game for the View.  If there is any other value used that isn't directly related to screen display (ie: a number of pixels) then you've crossed into game logic, and it needs to be baked into the Model through code in the Interactor.  

We start the drag action with `Node.startDragAndDrop()`.  This method returns a `Dragboard` which is an extension of `Clipboard`.  `Dragboard` adds support for a drag image, and a `TransferMode`.  The `TransferMode` can be `COPY`, `MOVE` or `LINK`.  Here, we're just going to use `COPY`.  

We're going to set the image to the basic counter image define in the `UnitType` Enum via `Dragboard.setImage()`.  We're not going to bother messing around with its location relative to the mouse pointer.

OK, so now we almost have a "drag" started...

## Drag Data

We need to put some data into our drag via its `Dragboard`.  Unfortunately, we have to put *something* into our `Dragboard` or the drag and drop won't work.  Even if we never use it.  And in this case we really don't need anything because we're defining the start and end in the same block of code, so we know how it all got started and don't need to shuttle data around in the `Dragboard`.  

For our drag and drop logic, we need two pieces of information; the `UnitType` and the target `Location`.  So we'll define both of those as values of `DataFormat`...

``` kotlin
val dataFormatLocation = DataFormat("location")
val dataFormatUnitType = DataFormat("unitType")
```

The data in the `Dragboard` is stored in a `Map`, keyed on `DataFormat` value.  Drag and drop is supposed to be interoperable between applications running on a desktop, so a lot of this structure is designed to facilitate that.  We're probably abusing the structure by using it to divide the data into different chunks and storing it in the `Map`, but it will still work just fine.  

So really all we've done here is define two keys that are compatible with the `Dragboard` data `Map`.  We can put anything we want into the `Map` values, as long as it's serializable.

Down the road, we'll probably want the `Tiles` to behave differently depending on the `UnitType` being dragged, so it won't hurt to include it now.  It satisfies the JavaFX need to have data, and isn't too complicated.

## DragDone

Like any other View `EventHandler` that needs to trigger real action, it's main function is to simply invoke a `Consumer`, passing it the information required.  

In this case the `Consumer` is a `BiConsumer<UnitType, Location>`, which is pretty clear in itself.  All we need to do the work, is the type of unit being deployed, and the `Location` of the `Tile` in which it was dropped.

# The Target Tile

Now, let's look at the target end of the process, the `Tile`.

``` kotlin
override fun build(): Region = createStackPane().apply {
      children.addAll(createTerrainSprite(), createCoordinateLabel(), createOccupyingCounter())
      onMouseClicked = EventHandler { clickHandler.run() }
      onDragEntered = EventHandler { pseudoClassStateChanged(dragOver, true) }
      onDragExited = EventHandler { pseudoClassStateChanged(dragOver, false) }
      onDragOver = EventHandler { it.acceptTransferModes(TransferMode.COPY) }
      onDragDropped = EventHandler {
         it.dragboard.setContent(mapOf(Pair(dataFormatLocation, model.locationProperty.value)))
         it.consume()
      }
   }
```
This is the `TileViewBuilder.build()` method, which we didn't look at in the section about the `Tiles` because it almost completely deals with drag and drop.   

The calls to `Node.setOnDragEntered()` and `Node.setOnDragExited()` simple toggle the `dragOver`

Here's the css entry that controls this:

``` css
.hex-tile:drag-over {
  -border-colour: blue;
}
```
When the `PseudoClass` is `true`, then the `Tile` border is `blue`.  

The call to `Node.setOnDragOver()` activates the `Tile` for drop event.  Basically, this is just a way for the `Tile` to tell the `DragEvent` that it's OK to drop onto the `Tile`.  JavaFX natively allows for the selection to be based on one or more of the three `TransferModes`, but we're only using `COPY` so that's all we need to enable.  

Finally, the `Node.setOnDragDropped()` call supplies the `EventHandler` that puts the target `Tile Location` into the `Dragboard`.  

That's it for the View side of things.  All in, about 20 lines of code to enable it.

# Performing the Drag and Drop Action

How does the counter get into the `Tile`?  That's all in the Interactors...

First, we need a data structure to pass the movement of a unit into and out of the `Tiles`.  Now, we're jumping the gun a little bit with the "out of" part, but it's going to get used in the very next article, and leaving it out now means that we'll just have to come back to it then.  So it's in here now:

``` kotlin
data class UnitMovement(val source: Location? = null, val destination: Location? = null, val unitType: UnitType) {}
```

This is pretty simple.  We have three values, the source and destination of the move, and then the `UnitType` that's being moved.  Both the `source` and `destination` fields are defined as `Location?`, which means that they are nullable.  In Kotlin, that means you treat them very much like `Optional` in Java.  Also, we've initialized them to `null`, so you don't have to specify them in the constructor.

Next, we'll look at the `SetupInteractor`:

``` kotlin
class SetupInteractor(val model: SetupModel, private val deploymentHandler: (UnitMovement) -> Boolean) {

    .
    .
    .

   fun deployCounter(unitType: UnitType, location: Location) {
      if (deploymentHandler.invoke(UnitMovement(destination = location, unitType = unitType))) {
         unitType.updateDeployedAmount(1)
      }
   }

   private fun UnitType.updateDeployedAmount(amount: Int) = when (this) {
      UnitType.NONE -> {}
      UnitType.HVYTK -> model.heavyTanksDeployed.value += amount
      UnitType.LGHTK -> model.lightTanksDeployed.value += amount
      UnitType.HWZR -> model.howitzersDeployed.value += amount
      UnitType.INFTY1 -> model.infantryDeployed.value += amount
      UnitType.INFTY2 -> {}
      UnitType.GEV -> model.gevsDeployed.value += amount
      UnitType.MSLTK -> model.missileTanksDeployed.value += amount
      UnitType.CPA -> model.commandPostsDeployed.value += amount
      UnitType.CT1 -> {}
   }
}
```
There's some Kotlin stuff to unpack here, so let's look at it.

First, in the constructor we have `deploymentHandler` declared as `(UnitMovement) -> Boolean`.  This is essentially the same as using `Function<UnitMovement,Boolean>` in Java.  It's a function that takes a `UnitMovement` and returns a `Boolean`.

Next, let's look at that weird function at the bottom.  It's called an "extension function", and allows us to add a method to another class without publicly adding the method inside the class.  In this case, we're using the fact that it's scoped inside the Interactor to allow access to the Model and update values in it.

Then we get to `deployCounter()`, which we are defining as a `UnitMovement` with an empty source and the destination our target `Location`.  We pass that movement off to the `GameInteractor` to do the actual work, and if it's successful then we update the number of this `UnitType` that have been deployed.

So how does the `GameInteractor` part work?

``` kotlin
fun handleUnitMovement(unitMovement: UnitMovement): Boolean {
    val taken = unitMovement.source?.run {
       model.consumeTile(this) { tileModel ->
          swapIfValid(tileModel, unitMovement.unitType, UnitType.NONE)
       }
    } ?: true
    val placed = unitMovement.destination?.run {
       model.consumeTile(this) { tileModel ->
          swapIfValid(tileModel, UnitType.NONE, unitMovement.unitType)
       }
    } ?: true
    return (taken && placed)
 }

 private fun swapIfValid(tileModel: TileModel, current: UnitType, newType: UnitType) = if (tileModel.occupierProperty.value == current) {
    tileModel.occupierProperty.value = newType
    true
 } else {
    false
 }
```
Let's look at `swapIfValid()` first.  It checks to make sure that the current occupier of the `TileModel` is what is expected, and if it is, then it swaps it with the new value and returns true.  If it isn't what was expected, then it returns false.

The method `handleUnitMovement()` potentially calls `swapIfValid()` twice.  Once to clear out the source `TileModel` and the other to populate the destination `TileModel`.  If either `TileModel` is null, it skips this step and returns `true` for it.

Finally, the whole method return the `AND` of both results.

## The Handler in GameModel

The final piece of this is `GameModel.consumeTile()`:

``` kotlin
fun consumeTile(location: Location, tileConsumer: (TileModel) -> Boolean): Boolean {
   return tileModels.find { tileModel -> tileModel.locationProperty.value.equals(location) }?.run {
      tileConsumer.invoke(this)
   } ?: false
}
```
This method takes a `Location` and a `Function<TileModel, Boolean>`.  It searches through `GameModel.tileModels` looking for a `TileModel` that has the specified `Location`.  Then it executes the supplied function against it and returns the result.  If it cannot find a `TileModel` with that `Location` it returns `false`.  

This is basically a way to perform arbitrary actions against an `TileModel` with a specified `Location` without needing to know anything about the structure of the storage of the `TileModels`.  

# And This is Reflected in the View

When the move is completed, two values will have changed in the Models (remember that the source is always `null` so far).

First, the `TileModel` that has the `Location` specified as the target will get updated, and it's `Occupier` property will changed.  This will be instantly reflected in the `Tile` on the screen, as it has an `ImageView` with its `Image` property bound to this value.

Secondly, the number of units available to be deployed will drop by one.  Once again, the `Label` with this value is bound to a property in the Model.
