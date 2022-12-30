---
excerpt: HexMap
hexmap: /assets/cybertanks/hexmap.png
permalink: /javafx/cybertanks/hexmap
---

![CyberTank]({{page.banner}})

# The Hex Map

The Hex Map itself is just a layout container for a collection of `Tiles`.  It has no special functions of its own, and merely needs to scale itself and present the `Tiles`.

Here's the `Builder` for the Hex Map:

``` kotlin
class HexMapViewBuilder(private val model: MainModel, private val tileClickHandler: Consumer<TileModel>) : Builder<Region> {

   override fun build(): Region {
      val pane = Pane().apply {
         children.addAll(model.tileModels.map { tileModel ->
            TileController(tileModel, tileClickHandler).getView().apply {
               translateXProperty().bind(Bindings.createDoubleBinding({ (tileModel.locationProperty.value.column - 1) * model.hexWidth.get() * 0.75 },
                                                                      model.hexWidth))
               translateYProperty().bind(Bindings.createDoubleBinding({ calculateYTranslate(tileModel.locationProperty.value) }, model.hexHeight))
            }
         })
         val numColumns: Int = model.tileModels.maxOfOrNull { tileModel -> tileModel.column } ?: 0
         val numRows: Int = model.tileModels.maxOfOrNull { tileModel -> tileModel.row } ?: 0
         minWidthProperty().bind(Bindings.createDoubleBinding({ model.hexWidth.get() * numColumns * 0.76667 + 5 }, model.hexWidth))
         minHeightProperty().bind(Bindings.createDoubleBinding({ (numRows + 0.5) * model.hexHeight.get() }, model.hexHeight))
         style = "-fx-background-color: teal;"
      }
      return StackPane(pane).apply {
         padding = Insets(5.0)
         maxWidthProperty().bind(pane.minWidthProperty().add(10))
         maxHeightProperty().bind(pane.minHeightProperty().add(10))
      }
   }

   private fun calculateYTranslate(location: Location): Double =
      ((location.row - 1) * model.hexHeight.get()) + (if (location.column % 2 == 0) model.hexHeight.get() / 2.0 else 0.0)
}

```
