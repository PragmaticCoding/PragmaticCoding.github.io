---
title:  "Constrained Properties"
date:   2025-08-01 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/constrained-properties
ScreenSnap0: /assets/elements/Constrained1.gif
ScreenSnap1: /assets/elements/Constrained2.gif
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
Listener: /assets/elements/ConstrainedListener.png
OLArticle: /javafx/elements/observable-classes-lists
OBGuide: /javafx/elements/observables_guide
GitHubLink: https://github.com/thomasnield/ConstrainedProperty

JavaDocOL: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/ObservableList.html
JavaDocFXC: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/FXCollections.html

excerpt: "What if you could have Properties that were able to evaluate new values against a constraint, and either refuse to change to a non-compliant value, or flag themselves as outside the constraint?"
---

# Introduction

Recently, I was taking another look at the "Validation" classes inside ControlsFX.  I have used ControlsFX, in the past, and in particular I've used the "Validation" classes.  ControlsFX is a nifty library, with a bunch of good ideas, but I've come to think that it is a bit antiquated in its approach.  Particularly in the context of building Reactive JavaFX applications.

The way that ControlsFX implements its Validators is by attaching them to `Controls` in the layout.  It even has a means to expand this use by giving you a hook to tell it how to find the `value Property` of a custom control.  But this means that it is, by this alone, oriented towards dealing with the issue of, "Is the data inside this `Control` valid?".  

But I have to wonder if that is really the right question?

Certainly, we care if the data is valid.  But do we need the context of "the data inside this `Control`"?

I don't think so.  Furthermore, this approach pushes the validation process into the domain of the View.  But isn't validation logic really better thought of as "Application/Business" logic?  I think it is.

This makes me think the whole idea of implementing validation as an extension of the layout is a major "Feature Envy" issue.

For sure, whatever the results of our validation, we want to have it reflected in our View somehow.  But that doesn't mean that the View should be responsible for the validation.

Can't we validate the data without involving the layout?

## Nomenclature

Before we go on, we need a short detour to talk about some terms.  

You're not going to see the words, "valid", "invalid", "validate", "validation", "invalidation" or "validating" used any more in this article with respect to decisions about whether or not the data inside a `Property` meets some criteria.

That's because JavaFX already has the idea of "valid" and "invalidation" as a key concept in the `Observable` classes.  In this context, it refers to whether a value in an `Observable` has been changed (or potentially changed) and the library needs to re-evaluate any other `Observables` that might be dependent on it.  When you change the value in an `Observable`, JavaFX automatically "invalidates" it.

We're going to stay away from those terms because we are going to use them in exactly that context in this article, and I don't want it to get confusing.

Instead, we are going to talk about "constraints" because I can't think of a better term.  Values can fall within constraints, or they can be outside of those constraints.  Data inside constraints is "Ok", data outside is "Not Ok".  This is a bit lame, but at least it's clear.

# The Goal

It's best to understand what we are really trying to achieve.

1. Outside the View
1. Self-contained
1. Compatible with all other `Observables`
1. Easy to use
1. Compatible with the spirit of "Reactive" JavaFX
1. Effectively replaces the "Validation" classes in ControlsFX

## How ControlsFX Works

It also helps to understand a little bit about how ControlsFX approaches this task, but I don't want to dwell on the details too much.

The core of the ControlsFX `Validation` package is `ValidationSupport`.  This serves as a registry of all of the `Validators` that have been added to `Controls` on the screen and controls the operation of the validations.  The `Validators` themselves contain some kind of decision making element (like a `Predicate`) and a message.  All of the public methods of `Validator` are static and involve creating or combining `Validators`.  

Validation results are communicated via `ValidationMessages` which identify the `Control`, a severity and a text message.  There is a class called `ValidationResult` which is a collection of `ValidationMessages` and contains methods for adding new messages and extracting messages.

The usual process is to add `Validators` to `Controls` via `ValidationSupport` and then monitor the `validationResult Property` of that `ValidationSupport` to determine what actions that your GUI needs to take to deal with any invalid data in any of the `Controls`.

Finally, the entire package is integrated with the ControlsFX Decorator package so that you can have automatic indications on the `Controls` that data is invalid.

PS:  ControlsFX has a severity called "ok".  So maybe my usage of it isn't quite that lame.

# ConstrainedProperty

My approach is to divorce the validation from the `Controls` and bake it into some custom `Property` classes.  Since in Reactive JavaFX we have virtually every element of the "State" of the GUI that we care about represented in the Presentation Model as `Properties`, this will still allow us to provide the same functionality as ControlsFX does without directly tying it to the layout.  

Most importantly, this means that we can think about constraints on the State elements without involving the layout.  Then we can have the layout link to the constraint results if we want.

## ConstrainedValue Interface

To begin with, we'll define an Interface to define the extra functionality of our `ConstrainedProperies`:

``` kotlin
interface ConstrainedValue<T> {
    val predicateProperty: ObjectProperty<Predicate<T>?>
    val valueOkProperty: ReadOnlyBooleanProperty
    fun isValueOk(): Boolean = valueOkProperty.value
    fun checkValue(newValue: T?): Boolean = newValue?.let { predicateProperty.value?.test(it) ?: true } ?: true
    fun getBean(): Any?
    fun getName(): String?
    fun getValue(): T?
}
```
I've called it `ConstrainedValue` because I don't want to imply `Property` characteristics like `Binding` that you'd expect if I called it `ConstrainedProperty`.  This is just the bare-bones of we need to make this work.

The idea with any Interface is that you could define some scope where a reference holds an instance of that Interface.  In this case, we would have a variable that refers to a `ConstrainedProperty` as a `ConstrainedValue`, and the methods that it has available are only the ones declared in that Interface.  You need to have enough methods to allow something meaningful to be done with that object.

We look at this more closely later.  For now though, you can note that these methods allow your client code to retrieve the current value, and to determine if a value is okay.  You can also manipulate the `Predicate` and identify the `ConstrainedProperty` via it's `bean` and `name`.  

## ConstrainedIntegerProperty

We'll start off by looking at a `ConstrainedIntegerProperty` because `Integer` is easy to think about.

``` kotlin
class ConstrainedIntegerProperty(
    initialValue: Int? = null,
    predicate: Predicate<Int>? = null,
    private val bean: Any? = null,
    private val name: String = ""
) : IntegerPropertyBase(), ConstrainedValue<Int> {

    override val predicateProperty = SimpleObjectProperty(predicate)
    private val valueOkPropertyImpl = ReadOnlyBooleanWrapper(this, "valueOK", false)
    override val valueOkProperty: ReadOnlyBooleanProperty
        get() = valueOkPropertyImpl.readOnlyProperty

    init {
        initialValue?.let { super.value = it }
        predicateProperty.subscribe(Runnable {
            testValue(value)
            fireValueChangedEvent()
        })
    }

    override fun set(newValue: Int) {
        testValue(newValue)
        super.set(newValue)
    }

    private fun testValue(newValue: Int?) {
        checkValue(newValue).let {
            if (it != valueOkProperty.value) fireValueChangedEvent()
            valueOkPropertyImpl.value = it
        }
    }

    override fun getBean(): Any? = bean

    override fun getName(): String? = name
}
```
This extends from `IntegerPropertyBase`, which is the same class that `SimpleIntegerProperty` extends from.  So this is essentially a drop-in replacement for `SimpleIntegerProperty`.  

There is storage for the predicate in an `ObjectProperty<Predicate>`, and storage for the status of the current value (whether or not it is within the constraints defined by the predicate) in a `BooleanProperty`.  The `BooleanProperty` is implemented as a private `ReadOnlyBooleanWrapper`, and exposed to the outside world as `ReadOnlyBooleanProperty` called `valueOkProperty`.  Once again, it would be nice call it something `validValueProperty`, but then we'd get confusion with "valid" as already defined for JavaFX `Observables`.

The constructor initializes everything and has default values for all parameters.  Default values for `bean` and `name` are given, which means that you can skip those parameters if you like without needing secondary constructors.  Also, `predicate` has a default value of `null`.  This means that you could use `ConstrainedIntegerProperty` instead of `SimpleIntegerProperty` absolutely everywhere, and just expose them as `IntegerProperty` and your client code won't know the difference because a `ConstrainedProperty` without a `Predicate` is just a `Property`.

The `init{}` block is somewhat like a constructor.  Just like with `SimpleIntegerProperty`, we check to see if the initial value is non-null and then call `super.setValue()`.  Then we add a `Subscription` to recheck the current `value` whenever the `predicate` is changed - because it might not be ok with the new `predicate`.

The `set()` method is overridden to add a call to `checkValue()` whenever the value is changed.  Even a `Binding` is still going to call `set()`, so there's no way that a change can slip through without being checked.  

In order to really check that the `value` is ok, we need a non-null `value` and a non-null `predicate`.  Otherwise we just set it to `true`.  If the `valueOkProperty` changes, then we invalidate the `ConstrainedProperty`.  We'll look at this later, but this allows you to create a `Binding` that returns a value which is effecitively dependent on the `valueOkProperty` but only has a formal dependency on the enclosing `ConstrainedProperty`.

## Using the ConstrainedProperty

Here's a sample application that will demonstrate how the `ConstrainedIntegerProperty` works:

``` kotlin
class ConstrainedExample0 : Application() {

    val messages = SimpleStringProperty("")
    val testInt = ConstrainedIntegerProperty(
        8,
        { it < 10 },
        100,
        "TestInt"
    )


    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 500.0, 300.0).addWidgetStyles()
        stage.title = "Constrained Property Example 0"
        stage.show()
    }

    private fun createContent(): Region = HBox(10.0).apply {
        val resultBinding = object : StringBinding() {
            init {
                super.bind(testInt)
            }

            override fun computeValue(): String? {
                println("Recomputing ${testInt.valueOkProperty}")
                return if (testInt.isValueOk()) {
                    "TestInt value of ${testInt.value} is Okay"
                } else {
                    "TestIne value of ${testInt.value} is Bad"
                }
            }
        }
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
                Label("getValue(): ") styleAs LabelStyle.PROMPT,
                Label().apply {
                    testInt.subscribe { _ -> text = testInt.value.toString() }
                }
            )
            children += HBox(
                10.0,
                Label("Data OK: ") styleAs LabelStyle.PROMPT,
                Label().bindTo(testInt.valueOkProperty.asString())
            )
            children += HBox(
                10.0,
                Label("Fluent: ") styleAs LabelStyle.PROMPT,
                Label().bindTo(testInt.multiply(2).asString())
            )
            children += Label().apply {
                textProperty().bind(resultBinding)
            }
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
It's an `HBox` holding two `VBoxes`.  The one on the left has two `Labels`, one bound to the current `value`, and the other bound to the `valueOkProperty`.  Then there are 4 `Buttons`.  The first two change the value to 4 and 20 respectively, and the other two change the `predicateProperty` to allow values less than 5 and 30 respectively.  These `Buttons` allow us to test every combination of changing values and Predicates that we need.

The right side contains a `TextArea` that is bound to the `StringProperty` called `messages`.  Each of the `Buttons` adds to `messages` so that we can see the moment that it was clicked.  There are listeners that display changes to the `valueOkProperty` and the `value`, and also when the `ConstrainedProperty` invalidates.

It's really important to look at `resultBinding`.  You can see that the sole dependency is `testInt`.

The value starts at 8, and the initial value of `predicateProperty` allows values less than 10.

When it's running, it looks like this:

![Screen Snap]({{page.ScreenSnap0}})

At the beginning the value is "8" and the limit is "< 10", so everything is good.  

The first `Button` click changes the limit to "5", and we see the messages indicating that the `valueOkProperty` is now `false`.  You can also see that the `Label` with the message has changed to "TestInt value of 8 is Bad", indicating that the `resultBinding` on `testInt` has fired.  

Then the "Value 4" `Button` is clicked, the `Property` invalidates, and the `valueOkProperty` goes back to true, invalidating the `Property`.  You can also see that `resultBinding` has recalculated.

Next, the "Value 20" `Button` is clicked, the `Property` invalidates, the `valueOkProperty` changes to `false`, and `resultBinding` recalculates.

And it goes on like that.  Each of the changes to either the Predicate or the value causes the `valueOkProperty` to be recalculated, and the entire `Property` invalidates, causing the `resultBinding` to also recalculate.

## Implementing Other Types

When I started looking at this I was determined that I would eliminate as much of the boilerplate as possible to make the implementation of each specific `ConstrainedProperty` type repeat as little code as possible.  Unfortunately, ever technique that I tried ended up added complexity that didn't seem worth the effort.

This means that `ConstrainedStringProperty` will look like this:

``` kotlin
class ConstrainedStringProperty(
    initialValue: String = "",
    predicate: Predicate<String>? = null,
    private val bean: Any? = null,
    private val name: String = ""
) : StringPropertyBase(), ConstrainedValue<String> {

    override val predicateProperty = SimpleObjectProperty(predicate)
    private val valueOkPropertyImpl = ReadOnlyBooleanWrapper(this, "valueOK", false)
    override val valueOkProperty: ReadOnlyBooleanProperty
        get() = valueOkPropertyImpl.readOnlyProperty

    init {
        initialValue?.let { super.value = it }
        predicateProperty.subscribe(Runnable {
            testValue(value)
            fireValueChangedEvent()
        })
    }

    override fun set(newValue: String) {
        testValue(newValue)
        super.set(newValue)
    }

    private fun testValue(newValue: String?) {
        checkValue(newValue).let {
            if (it != valueOkProperty.value) fireValueChangedEvent()
            valueOkPropertyImpl.value = it
        }
    }

    override fun getBean(): Any? = bean

    override fun getName(): String? = name
}
```
And this is logically identical to `ConstrainedIntegerProperty` but has every reference to `Int` changed to `String` and extends from `StringPropertyBase`.

All the other type-specific classes will look the same.  

# The Predicate Property

I've chosen to go with a `Predicate` enclosed in a `ObjectProperty` to remain consistent with classes like `FilteredList` that also use `ObjectProperty<Predicate>`.  You could achieve much of the same functionality with a `BooleanBinding` if you'd rather.

## Binding the Predicate Property

Because this is a `Property` you can bind it.  You'll need to bind it to some form of `ObjectBinding<Predicate>`, which, of course, returns a new `Predicate` when one of its dependencies changes.  Because of the `Subscription` on the `predicateProperty` in `ConstrainedProperty`, any change to the `Binding` is going to cause the `valueOkProperty` to be re-evaluated.

``` kotlin
class ConstrainedExample1 : Application() {

    val messages = SimpleStringProperty("")
    val testInt = ConstrainedIntegerProperty(
        8,
        { it < 10 },
        100,
        "TestInt"
    )
    val limit = SimpleIntegerProperty(10)


    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 500.0, 300.0).addWidgetStyles()
        stage.title = "Constrained Property Example 0"
        stage.show()
    }

    private fun createContent(): Region = HBox(10.0).apply {
        val resultBinding = object : StringBinding() {
            init {
                super.bind(testInt)
            }

            override fun computeValue(): String? {
                println("Recomputing ${testInt.valueOkProperty}")
                return if (testInt.isValueOk()) {
                    "TestInt value of ${testInt.value} is Okay"
                } else {
                    "TestInt value of ${testInt.value} is Bad"
                }
            }
        }
        val predicateBinding = object : ObjectBinding<Predicate<Int>>() {
            init {
                super.bind(limit)
            }

            override fun computeValue(): Predicate<Int> = Predicate { it < limit.value }
        }
        testInt.predicateProperty.bind(predicateBinding)
        testInt.predicateProperty.subscribe { _ -> messages.value += "Predicate changes -> Limit: ${limit.value}\n\n" }
        testInt.valueOkProperty.subscribe { newVal -> messages.value += "Ok Property now: $newVal \n" }
        testInt.subscribe(Runnable { messages.value += "TestInt invalidated \n" })
        testInt.subscribe { newVal -> messages.value += "Value changed to $newVal \n" }
        children += VBox(10.0).apply {
            minWidth = 240.0
            children += HBox(
                10.0,
                Label("CurrentValue: ") styleAs LabelStyle.PROMPT,
                Label().bindTo(testInt.asString())
            )
            children += HBox(
                10.0,
                Label("getValue(): ") styleAs LabelStyle.PROMPT,
                Label().apply {
                    testInt.subscribe { _ -> text = testInt.value.toString() }
                }
            )
            children += HBox(
                10.0,
                Label("Data OK: ") styleAs LabelStyle.PROMPT,
                Label().bindTo(testInt.valueOkProperty.asString())
            )
            children += HBox(
                10.0,
                Label("Upper Limit: ") styleAs LabelStyle.PROMPT,
                Label().bindTo(limit.asString())
            )
            children += Label().apply {
                textProperty().bind(resultBinding)
            }
            children += Spinner<Int>(2, 20, 10).apply {
                limit.bind(valueProperty())
            }
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
            alignment = Pos.CENTER
        } padWith 20.0
        children += TextArea().apply {
            textProperty().bind(messages)
            maxWidth = 300.0
        }
    }

}

fun main() {
    Application.launch(ConstrainedExample1::class.java)
}
```
Here we have added a `Spinner` to change the upper limit, and its `valueProperty` is bound to an `IntegerProperty` called `limit`.  Then we have a this `Binding` defined:

``` kotlin
val predicateBinding = object : ObjectBinding<Predicate<Int>>() {
    init {
        super.bind(limit)
    }

    override fun computeValue(): Predicate<Int> = Predicate { it < limit.value }
}
```
This is a `Binding` that returns a `Predicate<Int>` and is dependent upon the `limit IntegerProperty`.  So, whenever, `limit` changes, it will recalculate the `Predicate`.  This `Binding` is bound to the `testInt.predicateProperty`.  


It looks like this when it runs:

![Screen Snap]({{page.ScreenSnap1}})

# Suppressing "Bad" Values

I toyed with the idea of implementing something that would refuse to allow the `ConstrainedIntegerProperty` to change to a value not allowed by the `predicate`.  This is technically easy to do.  Just call `checkValue` first, and don't call `super.set()` if the result is `false`.

But...

What if the `predicate` changes, and the current `value` is no longer okay?

This opens a whole new can of worms.  There's no going back to a previous value - even if you stored it - because there's no guarantee that an previous value would fall within the constraints of the new `predicate`.  You could create some kind of object that had a `Predicate` and a default value, but then it starts to get even more complicated.

I looked at the idea of setting the `value` to `null` in this situation.  This raises the issue of "Null Safety" and whether or not the Fluent API" for `Bindings` would start to have issues.  It turns out that they don't.  

However, if instead of refusing to set the `value` to a "bad" value, what if you refused to *return* a bad value?

This seems to solve most of the issues.  Just change `getValue()` to this:

``` kotlin

```



# Notifications and a Constraining Framework

Remember `notifierProperty` in the very first version?  For the most part, there's nothing that you can do with `notifierProperty` that you couldn't do with a `Listener` or `Subscription` on `valueOkProperty`.  With one exception...

If put a `Listener` onto the `valueOkProperty` so that you can write some code that will give some feedback to the user when a value is invalid, that `Listener` is on the `valueOkProperty` and not the `ConstrainedProperty` that contains it.  You'll have to do something like this:

``` kotlin
  val ageProperty = ConstrainedIntegerProperty(0,{it > 29})
  constrainedProperty.valueOkProperty.addListener(ChangeListener {obVal, oldVal, newVal ->
     if (!newVal) {
        println("Age is less than 30")  
     }
  })
```
In other words, you have to define the `Listener` in the context of the containing `ConstrainedIntegerProperty`.  That's what the "Age is less" part of the `println()` statement does.  It adds the context that the `valueOkProperty` that has just flipped to false is inside the `ageProperty`.

This is fine, and can work.  But what if you want to keep to the spirit of the ControlsFX library, and split the messaging from the checking?  Now you have a problem.  Look at this diagram:

![Listener Diagram]({{page.Listener}})

That's why `valueOkProperty` was implemented the way it was in `ConstrainedPropertyBase`:

``` kotlin
val valueOkPropertyImpl = object : ReadOnlyBooleanWrapper(false) {
    override fun getBean(): ConstrainedProperty<T> {
        return wrapper
    }
}
override val valueOkProperty: ReadOnlyBooleanProperty
    get() = valueOkPropertyImpl.readOnlyProperty
```
In JavaFX, the `beans` are really supposed to represent the object that "owns" the `Property`, so it is useful for this purpose.  In this case, we've redefined `getBean()` to return the `wrapper` - which, of course, is the containing `ConstrainedProperty`.    

Now, how do we use this?

A lot of the work has already been done for us since JavaFX 19 with `Subscription`.  Let's look at the source code for `ObservableValue.subscribe()`:

``` java
default Subscription subscribe(Consumer<? super T> valueSubscriber) {
    Objects.requireNonNull(valueSubscriber, "valueSubscriber cannot be null");
    ChangeListener<T> listener = (obs, old, current) -> valueSubscriber.accept(current);

    valueSubscriber.accept(getValue());  // eagerly send current value
    addListener(listener);

    return () -> removeListener(listener);
}
```
`Subscription` is a functional interface with a bunch of default methods and one undefined method, `Subscription.unsubscribe()`.  Any implementation of `Subscription` just needs to supply that method - which is what this code does.  You can see that `subscribe()` takes a `Consumer` of the current value, and then creates a `ChangeListener` that simply calls `Consumer.accept()`.  The `unsubscribe()` method simply removes that `ChangeListener` from the `ObservableValue`.

This method returns the newly create `Subscription`, which means that we can hang on to it, and remove the `Listener` from the `ObservableValue` just by calling `Subscription.unsubscribe()`.  But there's nothing inside `Subscription` that will tell us what `Observable` the `Subscription` is associated with.  We can fix that by creating an class that implements it:

``` kotlin
class ConstrainedSubscription(
    val constrainedProperty: ConstrainedProperty<*>,
    val unsubscriber: () -> Unit
) : Subscription {
    override fun unsubscribe() {
        unsubscriber.invoke()
    }
}
```
The last thing we need to establish is how we are going to us this `ConstrainedSubscription` so we're going to define a `ConstrainedSubscriber`:

``` kotlin
typealias ConstrainedSubscriber = (constrainedProperty: ConstrainedProperty<*>, newValue: Boolean) -> Unit
```
This is basically the same as a Java `BiConsumer<ConstrainedProperty<*>, Boolean>`.

Here's how we would use it.

``` kotlin
fun ReadOnlyProperty<Boolean>.constrainedSubscribe(subscriber: ConstrainedSubscriber): ConstrainedSubscription? {
    if (this.bean is ConstrainedProperty<*>) {
        val theBean = this.bean as ConstrainedProperty<*>
        val listener: ChangeListener<Boolean> =
            ChangeListener { obs, old, current -> subscriber.invoke(theBean, current) }
        subscriber.invoke(theBean, value)
        addListener(listener)
        return ConstrainedSubscription(theBean) { removeListener(listener) }
    }
    return null
}
```
Why `ReadOnlyProperty`?

We need to be able to call `getBean()`, and `ReadOnlyProperty` is the highest up the hierarchy that we can go and still have this function.  It's not critical, since we're only ever going to call this on the `valueOkProperty`, but it's clean.

First thing, we can only call this on a `BooleanProperty` inside of a `ConstrainedProperty` of some type.  `ConstrainedPropertyBase.valueOkProperty` qualifies since it has that override on `getBean()` that returns `wrapper`.  If you call it on any other kind of `BooleanProperty` it will do nothing and return `null`.

The body of this method is just about the same as `ObservableValue.subscribe()`.  The `subscriber` takes two values, and the first is the `ConstrainedProperty` itself.  This means that the `subscriber` has full access to the enclosing `ConstrainedProperty` whenever it is invoked.  

Finally, we return a `ConstrainedSubscription` which also contains a reference to the enclosing `ConstrainedProperty`.  How does this help?

Now, we can do something that gets close to the ControlsFX `ValidationSupport`:

``` kotlin
abstract class ConstrainingFramework() {

    val subscriptions: ObservableList<ConstrainedSubscription> = FXCollections.observableArrayList()

    val subscriber: ConstrainedSubscriber =
        { constrainedProperty: ConstrainedProperty<*>, newValue: Boolean -> checkChange(constrainedProperty, newValue) }

    fun registerProperty(constrainedProperty: ConstrainedProperty<*>) {
        subscriptions += constrainedProperty.valueOkProperty.constrainedSubscribe(subscriber)
    }

    abstract fun checkChange(constrainedProperty: ConstrainedProperty<*>, valueOk: Boolean)
}
```
The most important thing here is that we have a generically defined `ConstrainedSubscriber` that we can apply to any `valueOkProperty` without modification.  


## Restricting the Value

This is something I pondered for a while.
