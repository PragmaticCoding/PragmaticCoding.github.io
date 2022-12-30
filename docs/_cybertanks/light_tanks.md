---
excerpt: Oops!  Forgot about Light Tanks, and they have a different unit cost.  How are we going to fix this?
permalink: /javafx/cybertanks/light_tanks
skip_link: true
---

![CyberTank]({{page.banner}})

# Oh-Oh!  Now We Have an Issue

(Repeated from the previous article)

When I did the original coding for this, it was a few months back, and I stopped because I felt that I needed to write the accompanying articles before the earliest bits of the programming were replaced with refactorings later on.  But I didn't have time to do the articles right away.  So, as much as I wanted to forge ahead with the coding, I stopped.

When I picked up the project and started writing the articles I noticed that I had left out the Light Tanks.  How could this be?  So i just went and added them in.  I created the images for the Light Tanks, and incorporated them into `CounterType` and put them in the Setup Sidebar.  Then when I was cleaning the code up to write this article I started thinking, "What's the unit cost for light tanks?"  By the specifications on the counter, they're way weaker than Heavy Tanks.

Oh darn!  They are half the cost of Heavy Tanks.  So now I don't just have Howitzers as the sole exception, now I have real variable costs.  

Let's look at what we need to do...

# Evolving Design

Before we start looking at the code, let's understand that this kind of thing happens all the time.  Requirements, scope and features get revealed as the project progresses.  You can't totally avoid this.  But you can defend against it...

YAGNI (You Ain't Gonna Need It)
: This is probably your best defence against changes in the design that happen through a project.  Every step of the way, make sure that you restrict yourself to just doing what you really need.  That way, when you do need to change something later on, you don't have to wade through all kinds of extra code that doesn't add any value.

Clean Coding
: This is the next best defence.  

In this particular case, Light Tanks are introduced in a later expansion game called "GEV".  Along with Light Tanks it also introduces more varied terrain types and some extra rules.  My intension was always to build a program that supported the later expansions, but I was working off the original rule book when I started programming.  Light Tanks aren't mentioned there, and they slipped through the cracks.  

Adding Light Tanks causes two issues...

Howitzers aren't a "one-off" exception any more
: As a one-off, you can get away with programming around a situation with some hard-coding and special case code blocks.  Once it becomes more common, it gets out of hand and you need to come up with a more integrated solution.

Light Tanks are 1/2 a Unit
: Oh snap!  The coding had been done based on the cost being an `Integer` and now there's no way around it, it will have to change to be a `Double`.  This is also going to change the logic a little bit.

# How to Store the Unit Cost

The unit cost is associated with the type of unit, so it's best to incorporate it in `CounterType`.  However, this transforms `CounterType` in its meaning.  It really was just info associated with the images of the counter and it made sense to call it that.  Even so, it was clear it was going to be used as a selector for `switch` statements to deal with unit types, so the name "CounterType" probably wasn't best.  At this point, though, it needs to change to `UnitType`.  From this point on, we'll call it `UnitType`.

I let Intellij Idea deal with that.  And changed any utility variables called `counterType` to `unitType` as part of the refactoring.  

Next, we need to add the unit cost...

``` kotlin
enum class UnitType(val label: String, private val resourceName: String, val unitCost: Double = 1.0) {
   NONE("None", ""),
   HVYTK("Heavy Tank", "HeavyTank"),
   LGHTK("Light Tank", "LightTank", unitCost = 0.5),
   HWZR("Howitzer", "Howitzer", unitCost = 2.0),
   INFTY1("Infantry 1", "Infantry1"),
   INFTY2("Infantry 2", "Infantry2"),
   GEV("Gev", "Gev"),
   MSLTK("Missile Tank", "MslTank"),
   CPA("CP Alpha", "CpAlpha"),
   CT1("CyberTank1", "cybertank1");

   companion object {
      private val infoImages = mutableMapOf<UnitType, Image?>()
      private val images = mutableMapOf<UnitType, Image?>()
      private fun createInfoImage(resourceName: String) =
         UnitType::class.java.getResource("/counters/${resourceName}Full.png")?.toExternalForm()?.let { Image(it) }

      private fun createImage(resourceName: String) =
         UnitType::class.java.getResource("/counters/black/${resourceName}.png")?.toExternalForm()?.let { Image(it) }

      private fun getInfoImage(unitType: UnitType): Image? {
         if (!infoImages.contains(unitType)) {
            infoImages[unitType] = createInfoImage(unitType.resourceName)
         }
         return infoImages[unitType]
      }

      private fun getImage(unitType: UnitType): Image? {
         if (!images.contains(unitType)) {
            images[unitType] = createImage(unitType.resourceName)
            println("Creating image: ${unitType.name} -> ${images[unitType]}")
         }
         return images[unitType]
      }

      fun byLabel(label: String) = values().find { it.label == label } ?: NONE
   }

   val infoImage: Image? by lazy { getInfoImage(this) }
   val image: Image? by lazy { getImage(this) }
}
```

Kotlin really shines here.  It was simple to add the new property, `unitCost` and I set it as a `Double` with a default value of `1.0`.  This way, I don't have to clutter up all of the constructor calls with the default value.  

For the two constructor calls with a non-zero `unitCost`, we can specify the parameter explicitly...

``` kotlin
LGHTK("Light Tank", "LightTank", unitCost = 0.5),
HWZR("Howitzer", "Howitzer", unitCost = 2.0),
```
I think this makes it really clear what's going on with these two `UnitType` values.  

So now we have a `Double` parameter that we can access via `UnitType.unitCost`.

Oops!  

Now that we've changed the name to `UnitType`, the "unit" in `unitCost` seems redundant.  Let's change it to `cost`.  That feels more like it makes sense.

# Changes to the SetupModel

All of the `Integer` based cost numbers in the Model need to change to `Double`.  Not the quantities of units, but just the cost stuff.  It's all at the bottom...

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
   val infantry: ObjectProperty<Int> = SimpleObjectProperty(30)
   val infantryDeployed: ObjectProperty<Int> = SimpleObjectProperty(0)
   val commandPosts: ObjectProperty<Int> = SimpleObjectProperty(1)
   val commandPostsDeployed: ObjectProperty<Int> = SimpleObjectProperty(0)

   val totalArmourAllowed: ObjectProperty<Double> = SimpleObjectProperty(20.0)
   private val _totalArmour: ObjectProperty<Double> = SimpleObjectProperty(0.0)
   val totalArmour: ObservableObjectValue<Double>
      get() = _totalArmour

   fun bindTotalArmour(binding: ObjectBinding<Double>) {
      _totalArmour.bind(binding)
   }
}
```
We just changed all of the `ObjectProperty<Int>` to `ObjectProperty<Double>`, and put a decimal point in the initial values.  Kotlin is actually a little bit stickier about the data value than Java.  Part of it has to do with the fact that you don't need the `<>` in the constructor call, it can figure it out if the data parameter is the right type.  

# In the ViewBuilder

Here it's all about the maximum values allowed in the Spinners, so we'll just look at how that code has changed:

``` kotlin
private fun createSpinnerBox(unitType: UnitType, boundProperty: ObjectProperty<Int>, deployedProperty: ObjectProperty<Int>) =
   HBox(12.0).apply {
      alignment = Pos.CENTER_LEFT
      val spinner = Spinner<Int>(0, 20, 0).apply { maxWidth = 60.0 }.apply {
         boundProperty.bind(valueProperty())
         (valueFactory as IntegerSpinnerValueFactory).maxProperty().bind(maxValueBinding(valueProperty(), unitType.cost))
      }
      children += listOf(createDragableImageView(unitType, boundProperty, deployedProperty), spinner, deployableLabel(boundProperty, deployedProperty))
      styleClass += "border-box"
   }

private fun maxValueBinding(valueProperty: ReadOnlyObjectProperty<Int>, unitCost: Double) =
      Bindings.createIntegerBinding({
                                      (((model.totalArmourAllowed.value - model.totalArmour.value) / unitCost) + valueProperty.value).roundToInt()
                                    },
                                    valueProperty,
                                    model.totalArmourAllowed,
                                    model.totalArmour)
```
The `Binding` still needs to be an `IntegerBinding` because the `Spinner` maximum needs to be an `Integer`.  The Kotlin standard library function `Double.roundToInt()` is nice and clean here.

Since we were already using `UnitType` an a parameter in the method call, we had the data we needed right at hand, so it didn't need a huge amount of refactoring.  We passed just the `UnitType.cost` value to `maxValueBinding` because that's the bare minimum we needed inside that method, and it was really just that one method call that changed.

# Now We're Back on Track

That's just about all it takes.  There was a little bit more, but that was in the Interactor, which we haven't looked at yet, so we'll see how it works there.  

It's important to understand that this change was actually pretty profound.  It changed the way that we view the Enum, and what it means and how it's used.  It didn't take a lot to make the change, mostly because there isn't a lot of code.  The point is, though, that if we keep to clean coding then there never should be a lot of code, and we shouldn't be scared of making these refactorings when they are needed.
