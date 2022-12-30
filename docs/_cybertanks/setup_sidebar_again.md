---
excerpt: After integrating Light Tanks, back to how the sidebar works.
permalink: /javafx/cybertanks/setup_sidebar_again
sidebar_image: /assets/cybertanks/setup_sidebar.png
---

![CyberTank]({{page.banner}})setup_sidebar

# How Setup Works

The main operation in the Setup Phase of the game is to allow the user to decide the composition of their forces, and where those forces will be placed at the beginning of the game.  In this article, we're going to concentrate on the selection of the units that the defender will be using.

The rules state that the defender is allowed 20 units of armour of any mix, with Howitzers counting as two each.  

# The Model

As usual, the best place to start is with the Model for the MVCI:

``` kotlin
class SetupModel {
   val heavyTanks: ObjectProperty<Int> = SimpleObjectProperty(0)
   val lightTanks: ObjectProperty<Int> = SimpleObjectProperty(0)
   val missileTanks: ObjectProperty<Int> = SimpleObjectProperty(0)
   val howitzers: ObjectProperty<Int> = SimpleObjectProperty(0)
   val gevs: ObjectProperty<Int> = SimpleObjectProperty(0)
   val heavyTanksDeployed: ObjectProperty<Int> = SimpleObjectProperty(0)
   val lightTanksDeployed: ObjectProperty<Int> = SimpleObjectProperty(0)
   val howitzersDeployed: ObjectProperty<Int> = SimpleObjectProperty(0)
   val missileTanksDeployed: ObjectProperty<Int> = SimpleObjectProperty(0)
   val gevsDeployed: ObjectProperty<Int> = SimpleObjectProperty(0)
   val totalArmourAllowed: ObjectProperty<Int> = SimpleObjectProperty(20)
   val infantry: ObjectProperty<Int> = SimpleObjectProperty(30)
   val infantryDeployed: ObjectProperty<Int> = SimpleObjectProperty(0)
   val commandPosts: ObjectProperty<Int> = SimpleObjectProperty(1)
   val commandPostsDeployed: ObjectProperty<Int> = SimpleObjectProperty(0)

   private val _totalArmour: ObjectProperty<Int> = SimpleObjectProperty(0)
   val totalArmour: ObservableObjectValue<Int>
   get() = _totalArmour

   fun bindTotalArmour(binding: ObjectBinding<Int>) {
      _totalArmour.bind(binding)
   }
}
```
It's not clear if this is the best possible structure.  Perhaps using some kind of a `Map` keyed on the `CounterType` Enum, with an object with a field for total and deployed would be better.  But for now we're only going to have these units, and although it's starting to look a little messy, it's not out of hand.

Note that we want `totalArmour` to be a read only value calculated via `Binding`.  But the calculation for the `Binding` is clearly game logic, so it has to go in the Interactor.  So we make the private `_totalArmour` field, and then expose the `ObservableObjectValue<Int>` through `totalArmour`, with the getter delegating to the private field.  Then we supply a method to allow the Interactor to create the `Binding`.  We'll see how that works in a bit.

As usual, we're using `ObjectProperty<Int>` instead of `IntegerProperty` because of the hierarchy differences, and I don't have to deal casts from `Number` to make stuff work.

# The ViewBuilder

![The Setup Sidebar]({{page.sidebar_image}}){:class="img-responsive" style="float: right" padding-right="20px"}This is what the sidebar looks like.  We have a box for each unit type, and for the armour counters we have a `Spinner` that allows the user to select how many of each unit are going to be used.  That `Binding` for total armour is going to come in to play, as it controls the maximum amount of the armour units that can be selected.

The images here are actually closer in appearance to the counters used in the game than the ones we'll use on the map.  They have the name of the type of unit, and some information about how it works in the game.  The "D1", "D2" or "D3" indicates the defensive power of the unit.  The "M0","M2","M3" and "M4-3" indicates the amount that the unit can move each turn.  You'll notice that Howitzers cannot move at all.  

Underneath the name of the unit, there's a pair of numbers in the format of "#/#".  This represents the attack capablity of the unit.  The first number is the attack power, and the second is the range.  

None of this information is particularly useful in terms of programming the setup phase, but it does allow you to see how the units differ and their strategic value.  

Let's look at the code:

``` kotlin
 class SetupViewBuilder(val model: SetupModel, private val deployHandler: BiConsumer<CounterType, Location>) : Builder<Region> {
   override fun build(): Region = VBox(2.0,
                                       createSpinnerBox(CounterType.HVYTK, model.heavyTanks, model.heavyTanksDeployed),
                                       createSpinnerBox(CounterType.LGHTK, model.heavyTanks, model.heavyTanksDeployed),
                                       createSpinnerBox(CounterType.GEV, model.gevs, model.gevsDeployed),
                                       createSpinnerBox(CounterType.MSLTK, model.missileTanks, model.missileTanksDeployed),
                                       createSpinnerBox(CounterType.HWZR, model.howitzers, model.howitzersDeployed),
                                       createFixedBox(CounterType.INFTY1, model.infantry, model.infantryDeployed),
                                       createFixedBox(CounterType.CPA, model.commandPosts, model.commandPostsDeployed)).apply {
      padding = Insets(4.0, 8.0, 0.0, 0.0)
   }

   private fun createSpinnerBox(counterType: CounterType, boundProperty: ObjectProperty<Int>, deployedProperty: ObjectProperty<Int>) = HBox(12.0).apply {
      alignment = Pos.CENTER_LEFT
      val slider = Spinner<Int>(0, 20, 0).apply { maxWidth = 60.0 }.apply {
         boundProperty.bind(valueProperty())
         (valueFactory as IntegerSpinnerValueFactory).maxProperty().bind(maxValueBinding(valueProperty()))
      }
      children += listOf(createDragableImageView(counterType, boundProperty, deployedProperty), slider, deployableLabel(boundProperty, deployedProperty))
      styleClass += "border-box"
   }

   private fun maxValueBinding(valueProperty: ReadOnlyObjectProperty<Int>, unitCost: Int) =
      Bindings.createIntegerBinding({
                                      ((model.totalArmourAllowed.value - model.totalArmour.value) / unitCost) + valueProperty.value
                                    },
                                    valueProperty, model.totalArmourAllowed, model.totalArmour)

   private fun createFixedBox(counterType: CounterType, totalProperty: ObjectProperty<Int>, deployedProperty: ObjectProperty<Int>) = HBox(12.0).apply {
      alignment = Pos.CENTER_LEFT
      children += listOf(createDragableImageView(counterType, totalProperty, deployedProperty), deployableLabel(totalProperty, deployedProperty))
      styleClass += "border-box"
   }

   private fun deployableLabel(totalProperty: ObjectProperty<Int>, deployedProperty: ObjectProperty<Int>) = Label().apply {
      textProperty().bind(Bindings.createStringBinding({ (totalProperty.value - deployedProperty.value).toString() }, totalProperty, deployedProperty))
      styleClass += "tank-text"
   }
}

val dataFormatLocation = DataFormat("location")
```

I've left out the code for `createDragableImageView` as we'll look at it in the next article.  For now, it's enough to know that it creates one of those counter images we saw just above here.  

As usual, there's not really a lot of code here.  The main `build()` routine just creates a `VBox` and populates it with one of two types of box.  One type has the Spinner to select the number of units, and the other just has a fix value for the number of units.  

The boxes themselves are fairly straight-forward, but there a couple of things to look at.  The first is that the maximum value for each `Spinner` depends on the total number of armour units that have been selected, with the Howitzers counting as two armour units each.  The Model has properties for the total number of units selected (counting Howitzers as two each) and for the total amount allowed.  This allows a binding to be created which dynamically changes the maximum values for all of the `Spinners` when any of the others is changed.

# Oh-Oh!  Now We Have an Issue

When I did the original coding for this, it was a few months back, and I stopped because I felt that I needed to write the accompanying articles before the earliest bits of the programming were replaced with refactorings later on.  But I didn't have time to do the articles right away.  So, as much as I wanted to forge ahead with the coding, I stopped.

When I picked up the project and started writing the articles I noticed that I had left out the Light Tanks.  How could this be?  So i just went and added them in.  I created the images for the Light Tanks, and incorporated them into `CounterType` and put them in the Setup Sidebar.  Then when I was cleaning the code up to write this article I started thinking, "What's the unit cost for light tanks?"  By the specifications on the counter, they're way weaker than Heavy Tanks.

Oh darn!  They are half the cost of Heavy Tanks.  So now I don't just have Howitzers as the sole exception, now I have real variable costs.  

Let's stop here, and then devote an entire article to refactoring for Light Tanks.
