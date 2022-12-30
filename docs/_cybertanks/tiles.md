---
excerpt: Tiles are the autonomous units of display in the Hex Map.  Let's see how they work.
hexmap: /assets/cybertanks/hexmap.png
permalink: /javafx/cybertanks/tiles
---

![CyberTank]({{page.banner}})

# Tiles - The Basic Unit of Display

Our hex map is made up of hexagonal tiles, and each tile is its own little MVCI construct with an hexagonal layout shape.

# The Tile Model

The model is pretty simple.  We need some data for the location of the `Tile` and it's size, the terrain type and feature, and who's occupying the `Tile`.

``` kotlin
class TileModel(location: Location, val width: ObservableDoubleValue, val height: ObservableDoubleValue) {
  val terrainFeatureProperty: ObjectProperty<Int> = SimpleObjectProperty(0)
  val terrainTypeProperty: ObjectProperty<TerrainType> = SimpleObjectProperty(TerrainType.NONE)
  val selectedProperty: BooleanProperty = SimpleBooleanProperty(false)
  val occupierProperty: ObjectProperty<CounterType> = SimpleObjectProperty(CounterType.NONE)
  val locationProperty: ObjectProperty<Location> = SimpleObjectProperty()
  val column: Int
  get() = locationProperty.value.column
  val row: Int
  get() = locationProperty.value.row

  init {
    this.locationProperty.set(location)
  }
}
```
We've also got a couple of convenience methods to pull the row and column out of the `Tile` location.  

Having the height and width come in as externally provided `Observables` means that we can rescale the entire map at once by changing those properties as the application level.

# The Tile ViewBuilder

All of the layout is handled by a single class `TileViewBuilder`.  Let's take a look at how it works

## CSS and SVG

In order to create the hexagonal shape, we're going to use the stylesheet parameter `-fx-shape`.  This allows you to specify an SVG path for the shape.  The size doesn't matter, as we'll scale the tile in the layout and it will just keep the shape.  

``` css
.hex-tile {
  -fx-shape: "M 300,150 225,280 75,280 0,150 75,20 225,20 z";
  -fx-border-color: -border-colour;
  -fx-border-width: 1px;
}
```


## The Layout

The basic layout is just a `StackPane`, we'll see how this makes sense in a little bit:

``` kotlin
private fun createStackPane() = StackPane().apply {
      styleClass.add("hex-tile")
      maxHeightProperty().bind(model.height)
      maxWidthProperty().bind(model.width)
      minHeightProperty().bind(model.height)
      minWidthProperty().bind(model.width)
      background = Background(BackgroundFill(model.terrainTypeProperty.value.colour, null, null))
      model.terrainTypeProperty.addListener { _ -> background = Background(BackgroundFill(model.terrainTypeProperty.value.colour, null, null)) }
   }
```
Here you can see the styleclass added, which gives it its shape, and then it's size is bound to the height and width as specified in the `TileModel`. Finally, the background is set to be a `BackgroundFill` with its colour set to that defined in the `TerrainType` Enum, and bound to the `TileModel.TerrainType`.
