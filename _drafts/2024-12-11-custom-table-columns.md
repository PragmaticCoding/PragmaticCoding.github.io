---
title:  "Type Specific TableColumns"
date:   2024-12-10 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/custom-table-columns
ScreenSnap0: /assets/elements/TableColumn0.png
ScreenSnap1: /assets/elements/TableColumn1.png
ScreenSnap2: /assets/elements/TableColumn2.png
ScreenSnap3: /assets/elements/TableColumn3.png
ScreenSnap4: /assets/elements/TableColumn4.png
ScreenSnap5: /assets/elements/TableColumn5.png
ScreenSnap6: /assets/elements/TableColumn6.png
ScreenSnap7: /assets/elements/TableColumn7.png
ScreenSnap8: /assets/elements/TableColumn8.png
ScreenSnap9: /assets/elements/TableColumn9.png
ScreenSnap10: /assets/elements/TableColumn10.png

Diagram: /assets/elements/ListProperties.png
part1: /javafx/elements/observable-classes-generics
part2: /javafx/elements/observable-classes-typed

excerpt: A look at the Observable classes that wrap ObservableList
---

# Introduction

If you rigorously follow DRY (Don't Repeat Yourself), and you should, then you'll eventually have to think about "Big Picture DRY".  This all about avoiding repeating yourself in different projects, and different parts of projects.  Not just within a single class or module of an application.

If you are one of those programmers that likes to use `TableView` - and there are lots of you out there - then one of the ways you might be repeating yourself without thinking about it is your `TableColumns`.  Or, and this is even more likely, you're avoiding properly configuring some kinds of `TableColumns` because it's just too much tedious work to do every time.

## Different Data Types Should be Presented Differently

`TableColumns` are easy to set up when the data in the column is just a simple `String`.  But what about numbers?  What about dates and money?  

Once you get into these questions, then you're suddenly looking at custom `TableCells` to handle the presentation and the difference between adding a `TableColumn` with nothing more than a `CellValueFactory` and doing it right seems huge.  

``` kotlin
class Example1 : Application() {
    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 400.0, 350.0).apply {
            Example1::class.java.getResource("example.css")?.toString()?.let { stylesheets += it }
        }
        stage.show()
    }

    private fun createContent(): Region = BorderPane().apply {
        center = TableView<TableData1>().apply {
            columns += TableColumn<TableData1, String>("Column A").apply {
                setCellValueFactory { it.value.value1 }
                styleClass += "a-column"
            }
            columns += TableColumn<TableData1, Double>("Temperature").apply {
                setCellValueFactory { it.value.value4.asObject() }
            }
            columns += TableColumn<TableData1, String>("Column C").apply {
                setCellValueFactory { it.value.value2 }
                styleClass += "c-column"
            }
            for (x in 1..30) items += TableData1(
                "A",
                "B",
                "C",
                Random.nextDouble(45.0)
            )
        }
        padding = Insets(10.0)
    }
}
```
Which looks like this:

![Very Ugly Table]({{page.ScreenSnap0}})

I think we can all agree that this is very, very ugly.  We can clean it up a bit by rounding it off to one decimal place by using `ObservableValue.map{}`:

``` kotlin
columns += TableColumn<TableData1, Double>("Temperature").apply {
    setCellValueFactory { it.value.value4.asObject().map { number -> Math.round(number * 10.0) / 10.0 } }
}
```

![Less Ugly Table]({{page.ScreenSnap1}})

Which is a little bit less ugly, but still not very good.  The worst thing is that the numbers are all left justified, so let's fix that by adding a style class selector `temperature-column` to the `TableColumn` and then fixing the alignment in the style sheet:

``` css
.table-row-cell .temperature-column  {
  -fx-alignment: center-right;
}
```

![Even Less Ugly Table]({{page.ScreenSnap2}})

One last problem is that the column is way too wide for the data because the heading, "Temperature" is a long word.  "Temp" would work but it's also commonly used for "temporary" (at least to programmers) and doesn't seem right.  Let's just use a graphic instead:

``` kotlin
columns += TableColumn<TableData1, String>("").apply {
    styleClass += "temperature-column"
    setCellValueFactory {
        it.value.value4.asObject().map { number -> Math.round(number * 10.0) / 10.0 }
            .map { number -> "%6.1f".format(number) }
    }
    Example1::class.java.getResource("thermometer24.png")?.toString()?.let { graphic = ImageView(it) }
}
```
Which looks like this:

![Table with Graphic Column Header]({{page.ScreenSnap3}})

At this point, I'm not pretending that this is a great UI design.  This article is about programming techniques, so we're looking for interesting things to do here.  

# Creating a Custom Column

For me, this small amount of code would be enough to trigger DRY the next time I created a "Temperature" column.  We can do it by creating a builder function or by creating a custom class extending `TableColumn`.  If we were going to stop here, then a builder would be the best option.  But we're not going to stop here, so we'll create a custom class, which will look like this to start:

``` kotlin
class TemperatureColumn<T>(extractor: (data: T) -> DoubleProperty) : TableColumn<T, Double>() {
    init {
        styleClass += "temperature-column"
        setCellValueFactory {
            extractor.invoke(it.value).asObject().map { number -> Math.round(number * 10.0) / 10.0 }
        }
        TemperatureColumn::class.java.getResource("thermometer24.png")?.toString()?.let { graphic = ImageView(it) }
    }
}
```
All of the `apply{}` code from before has been moved into `init{}` in our custom class.  In the `TableView` setup code we invoke our `TemperatureColumn` like this:

``` kotlin
columns += TemperatureColumn<TableData1> { it.value4 }
```
Once we've done this, our attitude to how we approach this new `TableColumn` changes in two ways:

* It's now worth doing things that we wouldn't have put in the effort for with a "one-off" implementation.
* It's now worth doing things really, really technically correct and configurable.

We'll see this in the first enhancement that we make...

## Styling a New Colour for Temperatures Below Freezing

Let's look at how to change the colour of the text to indicate temperatures at or below freezing.  This is best done through a Pseudo-class so that we can set the colour via the style sheet.  

Let's start with the style sheet:

``` css
.table-row-cell .temperature-column:freezing  {
  -fx-text-fill: cornflowerblue;
  -fx-font-weight: bold;
}
```
To do this, we'll need to create the `PseudoClass` which will mean creating a custom `TableCell`:

``` kotlin
class TemperatureTableCell<T> : TableCell<T, Double>() {
    companion object PseudoClasses {
        val freezingPseudoClass: PseudoClass = PseudoClass.getPseudoClass("freezing")
    }

    init {
        itemProperty().subscribe { newValue ->
            pseudoClassStateChanged(freezingPseudoClass, (newValue?.let { (it <= 0.0) } ?: false))
        }
        textProperty().bind(itemProperty().asString())
    }
}
```
Which looks like this:

![Column With Blue Freezing Temperatures]({{page.ScreenSnap4}})

Clearly, it is possible to just code in the colour change for freezing temperatures as part of the `TemperatureTableCell`, and that approach might be more expeditious.  But when you are building a custom `TableColumn` for re-use, you want to get it as close to "right" as you can.  And that means that you treat the colour as styling and you implement it so that it can be styled through the style sheets.

You may have guessed, given the formula, that this column is intended to display °C.  What if you need °F?

## Styling the Units

The first thing we need to do is to move the rounding feature out of the `TemperatureColumn` itself, and into the `TemperatureTableCell` because we don't want to be doing any conversions on previously rounded numbers.  Then we can introduce an `Enum` called `TemperatureUnit` and an `ObjectBinding<Double>` to convert between Celcius input and Farenheit and Kelvin output.

``` kotlin
class TemperatureColumn<T>(extractor: (data: T) -> DoubleProperty) : TableColumn<T, Double>() {

    val outputUnits: ObjectProperty<TemperatureUnit> = SimpleObjectProperty(TemperatureUnit.C)

    infix fun outputAs(newOutput: TemperatureUnit) = this.apply { outputUnits.value = newOutput }

    companion object PseudoClasses {
        val freezingPseudoClass: PseudoClass = PseudoClass.getPseudoClass("freezing")
    }

    init {
        styleClass += "temperature-column"
        setCellValueFactory {
            extractor.invoke(it.value).asObject()
        }
        TemperatureColumn::class.java.getResource("thermometer24.png")?.toString()?.let { graphic = ImageView(it) }
        setCellFactory { TemperatureTableCell() }
    }

    enum class TemperatureUnit { C, F, K }

    inner class TemperatureTableCell<T> : TableCell<T, Double>() {
        init {
            itemProperty().subscribe { newValue ->
                pseudoClassStateChanged(freezingPseudoClass, (newValue?.let { (it <= 0.0) } ?: false))
            }
            textProperty().bind(
                ConversionBinding(itemProperty(), outputUnits)
                    .map { rawValue -> rawValue?.let { Math.round(rawValue * 10.0) / 10.0 }?.toString() })
        }
    }

    class ConversionBinding(
        val celciusValue: ObjectProperty<Double>,
        val outputUnits: ObjectProperty<TemperatureUnit>
    ) : ObjectBinding<Double?>() {
        init {
            super.bind(celciusValue, outputUnits)
        }

        override fun computeValue(): Double? = when (outputUnits.value) {
            TemperatureUnit.K -> celciusValue.value?.let { it + 270.0 }
            TemperatureUnit.F -> celciusValue.value?.let { (it * 9 / 5) + 32 }
            else -> celciusValue.value
        }
    }

}
```
First off, everything has been moved inside the `TemperatureColumn` class, and `TemperatureTableCell` has been labled an "inner" class so that it can access fields from the outer `TemperatureColumn` class.  There is a `ConversionBinding` that is dependent on `Cell's item` and `TableColumn's outputUnits Properties` and which handles the conversion from Celcius to all three output formats.  The `text Property` of `TemperatureTableCell` is now bound to the `item Property` through this custom `Binding` and then further processed to limit it to one decimal place.  

The `outputAs()` method of `TemperatureColumn` has been written as an `infix` decorator to make it easier to configure from the layout code.  

In the layout, I've added a second `TemperatureColumn` so that we can have one in Celcius and one in Farenheit:

``` kotlin
columns += TemperatureColumn<TableData1> { it.value4 } outputAs TemperatureColumn.TemperatureUnit.F
columns += TemperatureColumn<TableData1> { it.value4 }
```
And it looks like this:

![Table with Farenheit and Celcius Colums]({{page.ScreenSnap5}})

You can see here that the `freezing` Pseudo-class is still calculated properly as it is done on the celcius value.  The conversios happens as `textProperty()` is updatad, so the internal value of `item` always remains in Celcius.

### Styling the Units Via CSS

``` kotlin
class TemperatureColumn<T>(extractor: (data: T) -> DoubleProperty) : TableColumn<T, Double>() {

    private val outputUnits: StyleableObjectProperty<TemperatureUnit> =
        SimpleStyleableObjectProperty(OUTPUT_UNITS_META_DATA, TemperatureUnit.C)

    infix fun outputAs(newOutput: TemperatureUnit) = this.apply { outputUnits.value = newOutput }

    companion object StylingStuff {
        val freezingPseudoClass: PseudoClass = PseudoClass.getPseudoClass("freezing")
        val OUTPUT_UNITS_META_DATA: CssMetaData<TemperatureTableCell<*>, TemperatureUnit> = object :
            CssMetaData<TemperatureTableCell<*>, TemperatureUnit>(
                "-wfx-temperature-unit",
                StyleConverter.getEnumConverter(TemperatureUnit::class.java)
            ) {
            override fun isSettable(styleable: TemperatureTableCell<*>) =
                !(styleable.outputUnits.isBound)

            override fun getStyleableProperty(styleable: TemperatureTableCell<*>) = styleable.outputUnits
        }

        private val cssMetaDataList =
            (TableColumn.getClassCssMetaData() + OUTPUT_UNITS_META_DATA) as MutableList

        fun getClassCssMetaData() = cssMetaDataList
    }

    init {
        styleClass += "temperature-column"
        setCellValueFactory {
            extractor.invoke(it.value).asObject()
        }
        TemperatureColumn::class.java.getResource("thermometer24.png")?.toString()?.let { graphic = ImageView(it) }
        setCellFactory { TemperatureTableCell(outputUnits) }
    }

}

class TemperatureTableCell<T>(val outputUnits: StyleableObjectProperty<TemperatureUnit>) :
    TableCell<T, Double>() {

    override fun getControlCssMetaData() =
        (super.getControlCssMetaData() + TemperatureColumn.getClassCssMetaData()) as MutableList

    init {
        itemProperty().subscribe { newValue ->
            pseudoClassStateChanged(freezingPseudoClass, (newValue?.let { (it <= 0.0) } ?: false))
        }
        textProperty().bind(
            ConversionBinding(itemProperty(), outputUnits)
                .map { rawValue -> rawValue?.let { Math.round(rawValue * 10.0) / 10.0 }?.toString() })
    }
}
```
Now, this is a little bit complicated.  We've turned `outputUnits` into a `StyleableObjectProperty` which will enable us to connect it to a style sheet.  This is done via something called `CssMetaData`, which really should be defined statically.  However, since it's static, the `CssMetaData` doesn't need to be defined in the class that uses it.  In this case, we are going to be using the same `Property` for each `TableCell` in the `TableColumn`, so we can define it in the custom `TableColumn` and then include it in the `CssMetaDataList` for the `TemperatureTableCell`.

One more twist on this is that `getCssMetaData()` is final in `TableCell`, but we have `getControlCssMetaData()` that we can use instead.

And now we can do this in the style sheet:

``` css
.temperature-column {
  -wfx-temperature-unit: "k";
}
```

## Adding More Styleable Properties
