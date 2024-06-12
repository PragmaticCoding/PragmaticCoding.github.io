---
title:  "Enumerated PseudoClasses"
date:   2024-06-04 00:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/enum-pseudoclass
Diagram: /assets/posts/MVCI.png
Modena: /assets/elements/modena.css
excerpt: What if you need to control the styling of a Node over a range of values?  How can you do that, and is there any way create a framework that you can use over and over again?
---

# Introduction

The other day, I was looking at questions on StackOverflow.com, and I came across a question with a lot of JavaFX code.  The OP was doing some complicated things - perhaps more complicated than necessary - including doing some styling manually in his code.

He had four states that he wanted to style: "active", "inactive", "highlighted" and "error".  I was thinking, as I looked at the code, that it must be easier to do this using stylesheets and PseudoClasses.

But PseudoClasses classes only work in an "on/off" fashion.  And the OP for this question was doing the same thing.  He had a set of `Boolean` flags, and big blocks of code that looked at the values of the flags, turning them on and off and applying different stylings according to the value.

It occurred to me that any time you've got a set of values where only one can be true, then you could probably handle it better with a Enum of some sort.  And I wondered how you would integrate that with the JavaFX PseudoClass system to handle dynamic styling based on a set of mutually exclusive values.

That's what we're going to look at here.

# PseudoClasses

What is a "PseudoClass"?

A PseudoClass is a way to provide temporary styling on a `Node` (usually) connected to a `BooleanProperty` in the layout code.  For instance, a `Button` can have a "disabled" status which is controlled through `Button.disabledProperty()`.  When you look through the CSS stylesheet, you'll find something equivalent to `.button: disabled {}`.  Whatever is set inside the `{}` is applied to the `Button` only when the `disabled` PseudoClass is applied to it.

There are a number of pre-defined PseudoClasses in JavaFX that are attached to various `Properties` in `Node` and its subclasses.  Besides `disabled`, there's `armed`, `selected`, `focused` and `default`, just to name a few.  You can even combine them together into something like `.button :armed :focused`.  Not all of theses PseudoClasses are available for every `Node` subclass.

Without customizing the code for any of these standard PseudoClasses, you can implement you own styling for them just by creating a stylesheet with an entry for `selector: pseudoclass` (using real values for `selector` and `pseudoclass`).

# Custom PseudoClasses

You can also create your own PseudoClasses, and the mechanism to connect JavaFX `Nodes` to your custom PseudoClass is simple, although badly explained in the JavaDocs.

The process is easy to describe:  Create an instance of `PseudoClass` with an associated CSS identifier (like "selected"), then call `Node.pseudoClassStateChanged()` for whatever `Node` you want, specifying that `PseudoClass` instance and with a `true` or `false`.

That's it.  

This code will do it:

``` kotlin
val myPseudoClass = PseudoClass.getPseudoClass("flashing")

myLabel.pseudoClassStateChanged(myPseudoClass, true)
```

Generally speaking, though.  You wouldn't do it this way.  First, `myPseudoClass` should probably by static, or at least a singleton - in Kotlin we'd put it inside a `companion object`.  In Java you'd make it `static final`.  

Secondly, you'll normally associate the control of PseudoClass with a `BooleanProperty` and then put a `Subscription` or a `Listener` on it to call `Node.pseudoClassStateChanged()`.  This would look like this:

``` kotlin
companion object {
  val myPseudoClass: PseudoClass = PseudoClass.getPseudoClass("flashing")
}
    .
    .
    .
model.someBooleanProperty.subscribe{ newVal ->
    myLabel.pseudoClassStateChanged(myPseudoClass, newVal)
}    
```

## From the JavaDocs

The JavaDocs are a bit bewildering because of the sample code included in the introduction.  We should probably look at them just to understand how it works:

``` java
public boolean isMagic() {
       return magic.get();
   }

   public BooleanProperty magicProperty() {
       return magic;
   }

   public BooleanProperty magic = new BooleanPropertyBase(false) {

       @Override protected void invalidated() {
           pseudoClassStateChanged(MAGIC_PSEUDO_CLASS, get());
       }

       @Override public Object getBean() {
           return MyControl.this;
       }

       @Override public String getName() {
           return "magic";
       }
   }

   private static final PseudoClass
       MAGIC_PSEUDO_CLASS = PseudoClass.getPseudoClass("xyzzy");
```
The first thing that not explained at all, is that this code needs to exist inside the `MyControl` class.  This means that this implementation is intended to be used inside a custom class that is a subclass of `Node` (because it calls `pseudoClassStateChanged()`).

The next thing that you need to understand is more complicated.  They are creating a `BooleanProperty` by extending `BooleanPropertyBase`, not `SimpleBooleanProperty`.  This means that they need to supply methods for `getBean()` and `getName()` because these are established as abstract methods in one of the superclasses of `BooleanPropertyBase`.  

`SimpleBooleanProperty` is a class that only adds some constructors and the functionality required for `getBean()` and `getName()`.  Since this `Property` is being instantiated and defined at the same time, they don't need the constructors from `SimpleBooleanProperty`, and the `Bean` and `Name` parts are also easy to define in this context.  

The other slightly confusing thing is that they've overriden `BooleanProperty.invalidated()`.  Essentially, this is the same as adding an `InvalidationListener` to the `Property`.  It's not a bad approach, and it's something that I've been calling an [Action Property](https://www.pragmaticcoding.ca/javafx/action-properties) since it's now a `Property` that also does something else - in this case trigger a PseudoClass state change.

The problem with this approach is that it requires extending some `Node` subclass to create a custom class, and then you're burdened with that custom class that you have to keep track of.

It's not necessary, and as we saw above it can be done with two lines of code using an external `BooleanProperty`.

# Multiple Exclusive States

What if you want to establish a styling state for a `Node` that doesn't just have two values?  By this I mean, instead of just `on` and `off`, you have `state1`, `state2`, `state3` and so on.  

PseudoClasses on work with `on` and `off`.  But you *can* establish a number of PseudoClasses, each of which can either be `on` or `off` and then combine them together somehow such that if one of them turns `on`, then the others all turn `off`.

The obvious structure for a data element that can have one of a discrete number of values is `Enum`.  How can we turn an `Enum` into a mechanism to control a set of PseudoClasses that work together as described above?  

We're going to look at this in both Kotlin and Java, since they are significantly different.

## Kotlin

In Kotlin, an `Enum` can implement an `Interface`.  We'll start there:

``` kotlin
interface PseudoClassSupplier {
    fun getPseudoClass(): PseudoClass
}
```
Just one function: `getPseudoClass()`.

Here's why we want to do this:

``` kotlin
fun Node.bindPseudoClassEnum(property: ObservableObjectValue<out PseudoClassSupplier>) {
    property.subscribe { oldValue, newValue ->
        oldValue?.let { pseudoClassStateChanged(it.getPseudoClass(), false) }
        newValue?.let { pseudoClassStateChanged(it.getPseudoClass(), true) }
    }
    property.value?.getPseudoClass()?.let { pseudoClassStateChanged(it, true) }
}
```

This is an extension function on `Node`, which means it will work for just about anything that you might possibly want to style via a PseudoClass.  We pass it an `ObservableObjectValue` that contains something that can supply `PseudoClassSupplier`.  This method adds a `Subscription` that supplies both the old and the new values of the `ObservableObjectValue`.  When the `Subscription` is triggered, we set the state of the old `PseudoClass` to `off` for this `Node`, and set it to `on` for the new value.  This is done with null safety checking since we have no control over whether or not the `ObservableObjectValue` is empty (or was empty).

Finally, we use the initial value in the `ObservableObjectValue` to turn the state to `on` for this `Node`, if it's populated.

How do we use this then?

Here's an example of an `Enum` that implements `PseudoClassSupplier` and can be used in our `ObservableObjectValue<out PseudoClassSupplier>`:

``` kotlin
enum class StatusPseudoClass : PseudoClassSupplier {
    NORMAL, WARNING, ERROR, FAILED;

    companion object PseudoClasses {
        val normalPseudoClass: PseudoClass = PseudoClass.getPseudoClass("normal")
        val warningPseudoClass: PseudoClass = PseudoClass.getPseudoClass("warning")
        val errorPseudoClass: PseudoClass = PseudoClass.getPseudoClass("error")
        val failedPseudoClass: PseudoClass = PseudoClass.getPseudoClass("failed")
    }

    override fun getPseudoClass() = when (this) {
        NORMAL -> normalPseudoClass
        WARNING -> warningPseudoClass
        ERROR -> errorPseudoClass
        FAILED -> failedPseudoClass
    }
}
```
The tricky thing here is that we want the instantiations of the `PseudoClasses` to be effectively static, so we need to use a `companion object` to contain them.  However, we cannot reference the `companion object` in the constructors of the `Enum` elements (I tried), so we need to handle it in the body of `getPseudoClass()`.  Hence the `when (this) {}` structure.  

The result is that the specifics about any particular implementation of this approach are contained neatly into a single `Enum` class.  Once it has been established, it's easy to replicate over and over again and the mechanics not in your layout code.  

Let's look at how to implement this:

``` kotlin
class EnumPseudoClassApp : Application() {
    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 380.0, 200.0).apply {
            EnumPseudoClassApp::class.java.getResource("test.css")?.toString()?.let { stylesheets += it }
        }
        stage.title = "PseudoClasses Wow!"
        stage.show()
    }

    private fun createContent(): Region = VBox(10.0).apply {
        val status: ObjectProperty<StatusPseudoClass> = SimpleObjectProperty()
        children += Label("This is a Label").apply {
            styleClass += "status-label"
            bindPseudoClassEnum(status)
        }
        val toggleGroup = ToggleGroup()
        children += HBox(10.0).apply {
            children += ToggleButton("Normal").apply {
                setOnAction { status.value = StatusPseudoClass.NORMAL }
                toggleGroup.toggles += this
            }
            children += ToggleButton("Warning").apply {
                setOnAction { status.value = StatusPseudoClass.WARNING }
                toggleGroup.toggles += this
            }
            children += ToggleButton("Error").apply {
                setOnAction { status.value = StatusPseudoClass.ERROR }
                toggleGroup.toggles += this
            }
            children += ToggleButton("Failed").apply {
                setOnAction { status.value = StatusPseudoClass.FAILED }
                toggleGroup.toggles += this
            }
        }
        padding = Insets(50.0)
    }
}

fun main() = Application.launch(EnumPseudoClassApp::class.java)
```
And it looks like this:



I put the actions into `ToggleButtons` in a `ToggleGroup` just make it clear when one has been selected.  The external `Property` would probably be in a Model somewhere, but here we just have the local variable, `status`.  The PseudoClasses are applied to a `Label` via `Label.bindPseudoClassEnum(status)`.  The actual changes to the `Property` are handled by the `ToggleButton` actions.

Perhaps the most important thing here is that the PseudoClass handling code *isn't* in the layout code.  It's basically "plumbing" and isn't particularly specific to this layout.  I could see a case for putting this `Enum` class itself away in a utility package somewhere so that it can be used over and over.  When you think about it, having the statuses "NORMAL", "WARNING", "ERROR" and "FAIL" (or some variation of this) is a pretty universal concept, and could be used over and over without modification.  

But it still looks like a lot of code in the layout, mostly because of all of the `Buttons`.  Let's change it a little.

``` kotlin
private fun createContent(): Region = VBox(10.0).apply {
    val amount: ObjectProperty<Int> = SimpleObjectProperty(-1)
    children += Label().apply {
        styleClass += "status-label"
        textProperty().bind(amount.map { "This is a Label: $it" })
        bindPseudoClassEnum(amount.map {
            when {
                it < 0 -> StatusPseudoClass.FAILED
                it == 0 -> StatusPseudoClass.NORMAL
                it < 3 -> StatusPseudoClass.WARNING
                else -> StatusPseudoClass.ERROR
            }
        })
    }
    children += Button("Add 1").apply {
        setOnAction { amount.value += 1 }
    }
    padding = Insets(50.0)
}
```
Everything else remains the same.

The multiple `Buttons` are gone, and now we just have a single `Button` that increments an `Integer Property`.  The `PseudoClass` is now connected to this `Integer Property` through a `Binding` that uses `ObservableValue.map{}` to convert it to a value of the `StatusPseudoClass Enum`.  The `map{}` uses a variant of `when` which is the Kotlin equivalent to Java's `switch()`.

The value starts at `-1` and as the `Button` is clicked it cyles from `FAILED` through all of the other values, and the colour of the `Label` changes.
