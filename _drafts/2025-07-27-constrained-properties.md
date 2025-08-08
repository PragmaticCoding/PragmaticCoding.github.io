---
title:  "Constrained Properties"
date:   2025-08-01 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/constrained-properties
ScreenSnap0: /assets/elements/ConstrainedProperty0.png
ScreenSnap1: /assets/elements/ConstrainedProperty1.png
ScreenSnap2: /assets/elements/ConstrainedProperty2.png
ScreenSnap3: /assets/elements/ConstrainedProperty3.png
ScreenSnap4: /assets/elements/ConstrainedProperty4.png
ScreenSnap5: /assets/elements/ConstrainedProperty5.png
ScreenSnap6: /assets/elements/ConstrainedProperty6.png
ScreenSnap7: /assets/elements/ConstrainedProperty7.png
ScreenSnap8: /assets/elements/ConstrainedProperty8.png
ScreenSnap9: /assets/elements/ConstrainedProperty9.png
ScreenSnap10: /assets/elements/ConstrainedProperty10.png

Diagram: /assets/elements/ListProperties.png
OLArticle: /javafx/elements/observable-classes-lists
OBGuide: /javafx/elements/observables_guide
GitHubLink: https://github.com/thomasnield/ConstrainedProperty

JavaDocOL: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/ObservableList.html
JavaDocFXC: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/FXCollections.html

excerpt: "What if you could have Properties that were able to evaluate new values against a constraint, and either refuse to change to a non-compliant value, or flag themselves as outside the constraint?"
---

# Introduction

Recently, I was taking another look at the "Validation" classes inside ControlsFX.  I have used ControlsFX, in the past, and in particular I've used the "Validation" classes.  ControlsFX is a nifty library, with a bunch of good ideas, but I've come to think that it is a bit antiquated in its approach.  Particularly in the context of building Reactive JavaFX applications.

ControlsFX implements its Validators by attaching them to `Controls`.  This means that it is, by this alone, oriented towards dealing with the issue of, "Is the data inside this `Control` valid?".  

But is that really the right question?

Certainly, we care if the data is valid.  But do we need the context of "the data inside this `Control`"?

I don't think so.  Furthermore, this approach pushes the validation process into the domain of the View.  But isn't validation logic really better thought of as "Application/Business" logic?  I think it is.

So I think the whole idea of implementing validation as an extension of the layout is a major "Feature Envy" issue.

For sure, whatever the results of our validation, we want to have it reflected in our View somehow.  But that doesn't mean that the View should be responsible for the validation.

Can't we validate the data without involving the layout?

## Nomenclature

Before we go on, we need a short detour to talk about some terms.  

You're not going to see the words, "valid", "invalid", "validate", "validation", "invalidation" or "validating" used any more in this article with respect to decisions about whether or not the data inside a `Property` meets some criteria.

That's because JavaFX already has the idea of "valid" and "invalidation" as a key concept in the `Observable` classes.  In this context, it refers to whether a value in an `Observable` has been changed (or potentially changed) and the library needs to re-evaluate any other `Observables` that might be dependent on it.

So, we're going to stay away from those terms because we are going to use them in that context in this article, and I don't want it to get confusing.

Instead, we are going to talk about "constraints" because I can't think of a better term.  Values can be within constraints, or they can be outside of constraints.  Data inside constraints is "Ok", data outside is "Not Ok".

# The Goal

It's best to understand what we are really trying to achieve.

1. Self-contained
1. Compatible with all other `Observables`



# First Implementation
We'll start off by looking at a `ConstrainedIntegerProperty` because `Integer` is easy to think about.

``` kotlin
class ConstrainedIntegerProperty(
    initialValue: Int? = null,
    predicate: Predicate<Int>? = null,
    private val name: String = "",
    private val bean: Any? = null,
    notifier: (Int, String, Any?) -> Unit = { a, b, c -> }
) :
    IntegerPropertyBase() {

    val predicateProperty: ObjectProperty<Predicate<Int>?> = SimpleObjectProperty(predicate)
    private val valueOkPropertyImpl = ReadOnlyBooleanWrapper(false)
    val valueOkProperty: ReadOnlyBooleanProperty
        get() = valueOkPropertyImpl.readOnlyProperty
    val notifierProperty: ObjectProperty<(Int, String, Any?) -> Unit> = SimpleObjectProperty(notifier)

    init {
        initialValue?.let { super.value = it }
        predicateProperty.subscribe(Runnable {
            fireValueChangedEvent()
            checkValue(getValue())
        })
    }

    override fun set(newValue: Int) {
        super.set(newValue)
        checkValue(newValue)
    }

    private fun checkValue(newValue: Int?) {
        val checkValue = newValue?.let { predicateProperty.value?.test(it) ?: true } ?: true
        if (!checkValue) {
            notifierProperty.value.invoke(newValue, name, bean)
            fireValueChangedEvent()
        }
        valueOkPropertyImpl.value = checkValue
    }

    override fun getBean(): Any? = bean

    override fun getName(): String? = name
}
```
This extends from `IntegerPropertyBase`, which is the same class that `SimpleIntegerProperty` extends from.  So this is essentially a drop-in replacement for `SimpleIntegerProperty`.  

There is storage for the predicate in an `ObjectProperty<Predicate>`, and storage for the status of the current value (whether or not it is within the constraints defined by the predicate) in a `BooleanProperty`.  The `BooleanProperty` is implemented as a private `ReadOnlyBooleanWrapper`, and exposed to the outside world as `ReadOnlyBooleanProperty` called `valueOkProperty`.  Once again, it would be nice call it something `validValueProperty`, but then we'd get confusion with "valid" as already defined for JavaFX `Observables`.

Finally, there is storage for a "Notifier" which is a `Consumer` that takes three values.  In Kotlin this is `(Int, String, Any?) -> Unit`.  It takes the current value of the `Property`, the value stored in `name`, and the value stored in `bean`.  Finally!!! A use for `getName()` and `getBean()`!

The constructor initialize everything and has default values for all parameters.  They are defined in order likely usage, since, in Kotlin, once you skip a parameter you have to invoke the constructor parameters by name (like `bean = 100`) for any later ones.  You cannot use the `notifier` without `bean` and `name`, so it comes last.  

The `init{}` block is somewhat like a constructor.  Just like with `SimpleIntegerProperty`, we check to see if the initial value is non-null and then call `super.setValue()`.  Then we add a `Subscription` to recheck the current `value` whenever the `predicate` is changed - because it might not be ok with the new `predicate`.

The `set()` method is overridden to add a call to `checkValue()` whenever the value is changed.  Even a `Binding` is still going to call `set()`, so there's no way that a change can slip through without being checked.  

In order to really check that the `value` is ok, we need a non-null `value` and a non-null `predicate`.  Otherwise we just set it to `true`. If it evaluates to false, then the `notifier` function is invoked.  Note that this will be invoked even if the value was previously false.

## Suppressing "Bad" Values

I toyed with the idea of implementing something that would refuse to allow the `ConstrainedIntegerProperty` to change to a value not allowed by the `predicate`.  This is technically easy to do.  Just call `checkValue` first, and don't call `super.set()` if the result is `false`.

But...

What if the `predicate` changes, and the current `value` is no longer ok?

This is opening a whole can of worms.  There's no going back to a previous value - even if you stored it - because there's no guarantee that an previous value would fall within the constraints of the new `predicate`.  You could create some kind of object that had a `Predicate` and a default value, but then it starts to get even more complicated.

I looked at the idea of setting the `value` to `null` in this situation.  But then you're into the whole issue of "Null Safety" and whether or not the Fluent API" for `Bindings` would start to have issues.  We'll take a look at this later.

Finally, I decided that the use case for this was probably not worth the effort -- for now.  It's up to whatever uses the `ConstrainedIntegerProperty` to decide how it wants to handle bad values.  As much as the intention is to have the `predicate` defined as business/application logic, the most logical use of this is to inform the user, through the GUI about the issue with the data.  

## Implementing the ConstrainedProperty

Here's a sample application that will demonstrate how the `ConstrainedIntegerProperty` works:

``` kotlin
class ConstrainedExample0 : Application() {

    val messages = SimpleStringProperty("")
    val testInt = ConstrainedIntegerProperty(
        8,
        { it < 10 },
        "TestInt",
        100,
        { value, name, bean -> messages.value += "Value: $value of $name:$bean outside constraint\n" })


    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 500.0, 300.0).addWidgetStyles()
        stage.title = "Constrained Property Example 0"
        stage.show()
    }

    private fun createContent(): Region = HBox(10.0).apply {
        testInt.valueOkProperty.subscribe { newVal -> messages.value += "Ok Property now: $newVal \n" }
        testInt.subscribe(Runnable { messages.value += "TestInt invalidated \n" })
        testInt.subscribe { newVal -> messages.value += "Value changed to $newVal \n" }
        children += VBox(10.0).apply {
            children += HBox(
                10.0,
                Label("CurrentValue: ") styleAs LabelStyle.PROMPT,
                Label().bindTo(testInt.asString())
            )
            children += HBox(
                10.0,
                Label("Data OK: ") styleAs LabelStyle.PROMPT,
                Label().bindTo(testInt.valueOkProperty.asString())
            )
            children += Button("Value to 4").apply {
                onAction = EventHandler {
                    messages.value += "\nValue 4 Button Clicked\n"
                    testInt.value = 4
                }
            }
            children += Button("Value to 20").apply {
                onAction = EventHandler {
                    messages.value += "\nValue 20 Button Clicked\n"
                    testInt.value = 20
                }
            }
            children += Button("Upper Limit 5").apply {
                onAction = EventHandler {
                    messages.value += "\nLimit 5 Button Clicked\n"
                    testInt.predicateProperty.value = Predicate { (it < 5) }
                }
            }
            children += Button("Upper Limit 30").apply {
                onAction = EventHandler {
                    messages.value += "\nLimit 30 Button Clicked\n"
                    testInt.predicateProperty.value = Predicate { (it < 30) }
                }
            }
            alignment = Pos.CENTER
        } padWith 20.0
        children += TextArea().apply {
            textProperty().bind(messages)
            maxWidth = 300.0
        }
    }

}

fun main() {
    Application.launch(ConstrainedExample0::class.java)
}
```
It's an `HBox` holding two `VBoxes`.  The one on the left has two `Labels`, one bound to the current `value`, and the other bound to the `valueOkProperty`.  Then there are 4 `Buttons`.  The first two change the value to 4 and 20 respectively, and the other two change the `predicateProperty` to allow values less than 5 and 30 respectively.

The right side contains a `TextArea` that is bound to the `StringProperty` called `messages`.  Each of the `Buttons` adds to `messages` so that we can see the moment that it was clicked.  There are listeners that display changes to the `valueOkProperty` and the `value`, and also when the `ConstrainedProperty` invalidates.  

Finally, the `notifierProperty` will simply add to `messages` so that we can see when and how it fires.

The value starts at 8, and the initial value of `predicateProperty` allows values less than 10.

When it's running, it looks like this:

![Screen Snap]({{page.ScreenSnap0}})

At the beginning the value is "8" and the limit is "< 10", so everything is good.  

The first `Button` click changes the `value` to "20", and we see the messages indicating that the `valueOkProperty` is now `false`.  We also see that the `notifier` has fired.

Then the "Value 4" `Button` is clicked, the `Property` invalidates, and the `valueOkProperty` goes back to true, invalidating the `Property`.

Next, the "Value 20" `Button` is clicked, the `Property` invalidates, the `notifier` fires (because 20 > 10) and the `valueOkProperty` changes to `false`.

After that, the "Upper Limit 30" `Button` is clicked, the `Property` invalidates and the `valueOkProperty` changes to true.

Finally, the "Upper Limit 5" `Button` is clicked, the `Property` invalidates, the `notifier` fires and the `valueOkProperty` changes to false.  You can see at this point that the "Current Value" `Label` says "20", and the "Data OK" `Label` says false.

# Removing BoilerPlate

The code for `ConstrainedIntegerProperty` works fine, but it's virtually all boilerplate and will need to be repeated if we make a `ConstrainedStringProperty`, or `ConstrainedBooleanProperty` or whatever.  Can we avoid too much repetition?

## Creating a ConstrainedProperty Interface

The first step is to create an interface that defines the characteristics that make a `ConstrainedProperty` different from any other `Property`:

``` kotlin
interface ConstrainedProperty<T> {
    val predicateProperty: ObjectProperty<Predicate<T>?>
    val valueOkProperty: ReadOnlyBooleanProperty
    val notifierProperty: ObjectProperty<(T, String, Any?) -> Unit>
    fun getBean(): Any?
    fun getName(): String?
}
```
Since this `Interface` has nothing to do with the regular `Property` we don't need to worry about special classes for `Integer` or `String`.  We can just define it generically.

In Kotlin, you can define fields that any implementing class will have to actually implement.  But any class that implements this `Interface` will automatically expose these fields publicly.  Which is ok in Kotlin, because fields in Kotlin are a bit different from those in Java.  I've also made the `getBean()` and `getName()` methods part of the interface, you'll see why...

## Implementing the Interface in a Base Class

I'm just going to give you the code for a `ConstrainedPropertyBase` that implements all of the elements of the `ConstrainedProperty` interface.  You'll see how it get's used shortly:

``` kotlin
class ConstrainedPropertyBase<T>(
    predicate: Predicate<T>?,
    notifier: (T, String, Any?) -> Unit,
    private val name: String,
    private val bean: Any?
) : ConstrainedProperty<T> {

    override val predicateProperty: ObjectProperty<Predicate<T>?> = SimpleObjectProperty(predicate)
    val valueOkPropertyImpl = ReadOnlyBooleanWrapper(false)
    override val valueOkProperty: ReadOnlyBooleanProperty
        get() = valueOkPropertyImpl.readOnlyProperty
    override val notifierProperty: ObjectProperty<(T, String, Any?) -> Unit> = SimpleObjectProperty(notifier)
    override fun getBean(): Any? = bean
    override fun getName(): String? = name

    fun checkValue(newValue: T?): Boolean {
        val checkValue = newValue?.let { predicateProperty.value?.test(it) ?: true } ?: true
        if (!checkValue) {
            notifierProperty.value.invoke(newValue, name, bean)
        }
        valueOkPropertyImpl.value = checkValue
        return checkValue
    }
}
```
All of these elements are implemented generically, but in the same manner as in our original `ConstrainedIntegerProperty`.  The `checkValue()` method now returns a `Boolean`.

## The new ConstrainedIntegerProperty

Here's how it all goes together:

``` kotlin
class ConstrainedIntegerProperty private constructor(
    private val base: ConstrainedPropertyBase<Int>,
    initialValue: Int?,
    private val name: String,
    private val bean: Any?,
) : IntegerPropertyBase(), ConstrainedProperty<Int> by base {

    constructor (
        initialValue: Int? = null,
        predicate: Predicate<Int>? = null,
        name: String = "",
        bean: Any? = null,
        notifier: (Int, String, Any?) -> Unit = { a, b, c -> }
    ) : this(ConstrainedPropertyBase<Int>(predicate, notifier, name, bean), initialValue, name, bean)

    init {
        initialValue?.let { super.value = it }
        predicateProperty.subscribe(Runnable {
            base.checkValue(get())
            fireValueChangedEvent()
        })
    }

    override fun set(newValue: Int) {
        if (base.checkValue(newValue)) {
            fireValueChangedEvent()
        }
        super.set(newValue)
    }
}
```
There's a couple of things here that need some explanation.

The first is in the class declaration.  It says, `ConstrainedProperty<Int> by base`, and `base` is one of the constructor parameters of type `ConstrainedPropertyBase<Int>`.  

What does this do?

This is "Class Delegation".  It means that all of the elements of the specified `Interface` are delegated to the object specified after the "by".  So in this case, the elements in `ConstrainedProperty` are going to be implemented by delegating them to `base`.

The next thing to understand is the private and public constructors.

One of the limitations of this delegation is that it can **only** delegate to objects specified as constructor parameters.  You cannot use internal fields.  By using a private main constructor, this hides the `base` object from the outside world.

The secondary constructor calls the main constructor, creating a `ConstraintedPropertyBase` from its parameters.

The rest of the body of the class are functions that directly refer to elements of `Property` and cannot be delegated to `base`.  This is `set()`, `get()` and `fireValueChangedEvent()`.

To create a `ConstrainedStringProperty` we literally just need to change all of the references to `Int` to be `String` and extend from `StringPropertyBase`:

``` kotlin
class ConstrainedStringProperty private constructor(
    private val base: ConstrainedPropertyBase<String>,
    initialValue: String?,
    private val name: String,
    private val bean: Any?,
) : StringPropertyBase(), ConstrainedProperty<String> by base {

    constructor (
        initialValue: String? = null,
        predicate: Predicate<String>? = null,
        name: String = "",
        bean: Any? = null,
        notifier: (String, String, Any?) -> Unit = { a, b, c -> }
    ) : this(ConstrainedPropertyBase<String>(predicate, notifier, name, bean), initialValue, name, bean)

    init {
        initialValue?.let { super.value = it }
        predicateProperty.subscribe(Runnable {
            base.checkValue(get())
            fireValueChangedEvent()
        })
    }

    override fun set(newValue: String) {
        if (base.checkValue(newValue)) {
            fireValueChangedEvent()
        }
        super.set(newValue)
    }
}
```
