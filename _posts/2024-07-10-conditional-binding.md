---
title:  "Conditional Bindings"
date:   2024-07-20 00:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
taskArticle: /javafx/elements/fxat
nonbinaryArticle: /javafx/nonbinary-pseudoclass
permalink: /javafx/elements/conditional-binding
Modena: /assets/elements/modena.css
excerpt: A look at how to extend the idea of Bindings to include internal state to encapsulate the mechanics of more complicated relationships between ObservableValues in your GUI and your Model.
---

# Introduction

An interesting question come up on the Reddit JavaFX subreddit recently.  Someone was having issues with capturing the value from a `ComboBox` once the user had "locked in" their selection by hitting `<Enter>` or tabbing away from the `ComboBox`.  

The key difficulty is that `ComboBox` doesn't have any explicit signal for a "locked-in" selection.  There's no `ActionEvent` fired at lock-in, and no separate `Property` to hold a locked-in value.

So, how do you tell when a value has been locked-in?

It turns out that the visibility of the pop-up `ListView` of the `ComboBox` is the determiner.  If the pop-up `ListView` (let's just call it the "pop-up" from now on) is showing, then the value hasn't been locked in.  If they tab away or hit `<Enter>` then the pop-up disappears and the value stays the same and can be considered "locked-in".  

The question is, how do you differentiate in your application between a transient editing value and the locked in value when you are binding this value to the rest of your GUI or to your Model?  

This is problematic because, if you have your GUI dynamically responding to changes in the selected value - perhaps making different parts of your GUI visible or invisible, or changing the content of `Labels` - you might not want these changes to happen due to transient, un-locked-in, values in the `ComboBox`.   

## Actions and Values

In general, you have three mechanisms to make your JavaFX GUI respond dynamcially:

Actions
: These are usually invoked through `Events` which are "fired" in certain conditions.  Actions are good when you want to capture, "something happened", and are implemented via `EventHandlers`.

Values
: JavaFX values are usually enclosed in `Observable` wrapper classes that allow other elements of the GUI to respond to changes in their value.  The main mechanism to implement this approach is `Bindings`.

Listeners
: It is also possible to turn a change in the value contained in an `Observable` object into an action.  This is done via a `Listener` or a `Subscription`.

I think it's important to understand that there are some things that are fundamentally "actions" just by their nature.  When a user clicks on a `Button`, that's definitely an action and is probably best treated in your application as an action.  

But there are lots of other things that *might* be actions but also *might* be just a data change.  A user clicking on a `CheckBox` is an example of this.  Yes, there's clearly a user action taking place, but the main result might simply be that the `selectedProperty()` of the `CheckBox` has changed.  

The point is that you need to be intentional in your programming.  If you need to run some business logic when the user clicks on a `CheckBox` then you can put an `EventHandler` on the `Checkbox's onAction Event`.  However, if you take this approach, then your business logic won't run if you change the value in the `CheckBox` prgrammatically, or through a bidirectional Binding.  That may be okay if your *intention* is to respond to a user interaction with the GUI.  If you need to run your business logic whenever the value in the `CheckBox` changes, then maybe a `Subscription` is a better idea.

However, capturing an `Event` to trigger an action that simply copies data from one `ObservableValue` to another is probably going to be the wrong approach.  In those cases, you're better off using a `Binding` to link the two `ObservableValues` together.

But sometimes, the connection between the two `ObservableValues` seems to be too complicated to use a `Binding`.  And that's what we're going to talk about here...

# The ComboBox Scenario

Before we talk about solutions, we need to see the problem in action and understand what causes it.  So here's a simple application that demonstrates the problem:

``` kotlin
class ComboBoxTest1 : Application() {
    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 400.0, 400.0)
        stage.show()
    }

    private fun createContent(): Region = VBox(20.0).apply {
        val model = Model()
        children += Label().apply {
            textProperty().bind(model.testValue.map { "The current value in the ComboBox: $it" })
        }
        children += ComboBox(model.valueList).apply {
            model.testValue.bind(valueProperty())
        }
        padding = Insets(40.0)
    }
}


class Model {
    val testValue = SimpleObjectProperty<String>("")
    val valueList: ObservableList<String> =
        FXCollections.observableArrayList("abc", "abj", "def", "hij", "mno", "xyz", "123", "456")
}


fun main() = Application.launch(ComboBoxTest1::class.java)
```
The `textProperty()` of the `Label` is bound to the `testValue Property` of the Model which is bound to the `valueProperty()` of the `ComboBox`.  

The result looks like this:

{% include video id="vj96zSHzCiE" provider="youtube" %}

You can see how the value in the `Label` changes as the user scrolls through the values using the up and down keys.  But, as we've already discussed, these "values" while the pop-up is expanded are transient and you might not want them to cause dynamic changes to your GUI.

# Detecting the "Final" Value

There are two ways to make a selection in a ComboBox.  When you use the keyboard and hit `<Enter>` or click on a selection with the mouse.

Unfortunately, there is no `Property` in `ComboBox` that says, "The selection has been confirmed by the user".  Nor is there an `Event` that is fired that says, "The selection has been confirmed by the user".

What does happen, however, is that the pop-up `ListView` disappears.  And there are two `Events` for that; `onHiding` and `onHidden`.  The first fires just before it hides the pop-up, and the second fires just after it hides the pop-up.  

This means that we *can* capture something related to the user confirming their selection.  So let's try that:

``` kotlin
class ComboBoxTest2 : Application() {
    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 400.0, 400.0)
        stage.show()
    }

    private fun createContent(): Region = VBox(20.0).apply {
        val model = Model2()
        children += Label().apply {
            textProperty().bind(model.testValue.map { "The current value in the ComboBox: $it" })
        }
        children += ComboBox(model.valueList).apply {
            setOnHidden { model.testValue.value = value }
        }
        padding = Insets(40.0)
    }
}


class Model2 {
    val testValue = SimpleObjectProperty<String>("")
    val valueList: ObservableList<String> =
        FXCollections.observableArrayList("abc", "abj", "def", "hij", "mno", "xyz", "123", "456")
}


fun main() = Application.launch(ComboBoxTest2::class.java)
```
This looks like it works:

{% include video id="A55mPcAlbR8" provider="youtube" %}

# There is a Problem With This Approach

The origin of the problem is that `ComboBox` doesn't really support the idea of a "locked-in" selection.  The idea that if the pop-up is expanded, then user is still in the process of making a selection is a bit of a kludge.  

This approach seems to work.  However, look at this:

{% include video id="49sKadCt9T0" provider="youtube" %}

Oh no!

It turns out that the user can use the up and down arrow keys to change the selection without expanding the pop-up.  In this case, we have no `onHidden Event` to capture the lock-in, and we never update our Model with the selected value.  Additionally, if we change the value programmatically while the pop-up is closed, then it won't update the Model either.

This points out a fundamental difference between programming to events vs programming to state.  Events can be missed- or never happen - and then your data can get out of sync.  Programming to state doesn't have this problem.

So how can we turn this into a "State" approach?

`ComboBox` gives us two `Properties` that we can use: `showing` and `value`.  We only want to update our Model when either one of the two situations occurs:

1. When the `value` changes while `showing` is false.
1. When `showing` changes from `true` to `false`.

We can do this with two `Subscriptions`, one on each `Property`:

## Multiple Subscriptions

Here's the code:
``` kotlin
class ComboBoxTest3 : Application() {
    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 400.0, 400.0)
        stage.show()
    }

    private fun createContent(): Region = VBox(20.0).apply {
        val model = Model3()
        children += Label().apply {
            textProperty().bind(model.testValue.map { "The current value in the ComboBox: $it" })
        }
        children += ComboBox(model.valueList).apply {
            showingProperty().subscribe { newVal -> if (!newVal) model.testValue.value = value }
            valueProperty().subscribe { newVal -> if (!isShowing) model.testValue.value = newVal }
        }
        padding = Insets(40.0)
    }
}


class Model3 {
    val testValue = SimpleObjectProperty<String>("")
    val valueList: ObservableList<String> =
        FXCollections.observableArrayList("abc", "abj", "def", "hij", "mno", "xyz", "123", "456")
}


fun main() = Application.launch(ComboBoxTest3::class.java)
```

And this works as you would expect:

{% include video id="lXiFAlgl2aI" provider="youtube" %}

The biggest problem that I have with this approach is that it *overtly* takes state changes and turns them into actions in order to update the Model.  Surely this is something better done with a `Binding`.

The next issue that I have with this approach is that it needs to be defined in a scope that has access to both the Model and the properties of the `ComboBox`.  You could, however, put it in a method and pass the Model field, and the two `ComboBox Properties` to it.

Finally, it's really hard to generalize this, and it virtually needs to be defined as part of the layout.  It's also hard to create the relationship between the `value` and `showing` without having a Model to connect them to.

Maybe there's a better way to do this...

## Creating a Conditional Binding

It is possile to create a `Binding` that has dependencies on two different `ObservableValues`.  As a matter of fact, we do this all the time, and it's easy to do.  The problem is that `Bindings` don't have any way to say "Don't change anything" in their output.  So, if the `value` property changes while `showing` is true then how do we say, "Do nothing"?

Here's how:

``` kotlin
class ConditionalBinding<T>(
    private val dependency: ObservableValue<T?>,
    private val condition: ObservableBooleanValue
) : ObjectBinding<T?>() {

    init {
        super.bind(dependency, condition)
    }

    private var oldValue: T? = null

    override fun computeValue(): T? {
        if (condition.value) {
            oldValue = dependency.value
        }
        return oldValue
    }
}
```
Let's look at how this works...

First, we've introduced an internal field, `oldValue` of the same type as the generic type of the `ConditionalBinding`.  

Secondly, in the `computeValue()` method, which is the method that we're used to using to create custom `Binding` classes, we have some logic that updates `oldValue` based on the two conditions described above.

Finally, `computeValue()` will always return whatever is in `oldValue`, regardless of whether or not it has been updated.

And we can use it like this:

``` kotlin
class ComboBoxTest4 : Application() {
    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 400.0, 400.0)
        stage.show()
    }

    private fun createContent(): Region = VBox(20.0).apply {
        val model = Model4()
        children += Label().apply {
            textProperty().bind(model.testValue.map { "The current value in the ComboBox: $it" })
        }
        children += ComboBox(model.valueList).apply {
            model.testValue.bind(ConditionalBinding(valueProperty(), showingProperty().not()))
        }
        padding = Insets(40.0)
    }
}

class Model4 {
    val testValue = SimpleObjectProperty<String>("")
    val valueList: ObservableList<String> =
        FXCollections.observableArrayList("abc", "abj", "def", "hij", "mno", "xyz", "123", "456")
}


fun main() = Application.launch(ComboBoxTest4::class.java)
```
This functions exactly the same way that the two `Subscription` version does.  

However, you can see that the mechanics of it are completely hidden from the layout, and that we can instantiate the `ConditionalBinding` without any reference to the `Observable` that it will be bound to - just like any other `Binding`.  In fact, we could instantiate to a variable without binding it to anything at all - from that point on we could treat it like any other `ObservableValue`.

Furthermore, now that we've decided that this is how we want to handle `ComboBox`, we can put this stuff into a library and forget all about it, like this:

``` kotlin
fun <T : Any> ComboBox<T>.getConditionBinding() = ConditionalBinding(valueProperty(), showingProperty().not())

infix fun <T : Any> ComboBox<T>.conditionallyBind(boundProperty: ObjectProperty<T>) = apply {
    boundProperty.bind(getConditionBinding())
}
```
These are extension functions, which in Kotlin allow us to add new methods to an existing class without formally extending it.  In this case, we're adding two new functions to `ComboBox`, but you could do the same thing in Java by creating static methods that accept a `ComboBox` as a parameter.  Kotlin just makes it easier and cleaner.

The first extension function just creates a `ConditionalBinding` based on the `ComboBox`, and the other is a decorator that creates a `ConditionalBinding` and binds it to a `Property`.  The `infix` notation allows us to use it without enclosing the parameter in brackets and without prefixing it with a `.`.  

Now our code looks like this:

``` kotlin
class ComboBoxTest5 : Application() {
    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 400.0, 400.0)
        stage.show()
    }

    private fun createContent(): Region = VBox(20.0).apply {
        val model = Model5()
        children += Label().apply {
            textProperty().bind(model.testValue.map { "The current value in the ComboBox: $it" })
        }
        children += ComboBox(model.valueList) conditionallyBind model.testValue
        padding = Insets(40.0)
    }
}

class Model5 {
    val testValue = SimpleObjectProperty<String>("")
    val valueList: ObservableList<String> =
        FXCollections.observableArrayList("abc", "abj", "def", "hij", "mno", "xyz", "123", "456")
}


fun main() = Application.launch(ComboBoxTest5::class.java)
```
You can see here that we've completely removed all of the "plumbing" related to the `Binding` out of the layout code.  We have a library that now includes a custom `Binding` class, and two extension functions that make any `ComboBox` that we use work as close as we can to the way that we expect it to work without customizing `ComboBox` itself.  And it doesn't clutter up our layout at all.

# Bindings With State

If you are like me, you've never thought of `Bindings` as anything more than connective tissue to join together various `ObservableValues` in your GUI and your Model.  

But this `ConditionalBinding` is different.  It doesn't just say, "Manipulate these values as they change to create a new changing value".  

It has it's own, hidden, "State".  That's the `oldValue` field.  And this changes everything.

Let's be clear, this is still a `Binding` and, as such, it's job remains to respond to changes in its dependencies and *potentially* calculate a new value in response to changes in those dependencies.  But now you can do different things.

## A Change Counter

Let's try adding an extra `Label` that shows how many times the value in the `ComboBox` has had locked-in changes.  We can create a custom `Binding` like this:

``` kotlin
class CountingBinding<T>(private val dependency: ObservableObjectValue<T?>) : IntegerBinding() {

    init {
        super.bind(dependency)
    }

    private var oldValue: T? = dependency.value
    private var counter = 0

    override fun computeValue(): Int {
        if (oldValue != dependency.value) {
            oldValue = dependency.value
            counter++
        }
        return counter
    }
}
```
We have to be careful here.  The `Binding` will potentially recompute every time that the dependencies are invalidated - not every time that it changes.  So if we want to track changes, then we need to weed out the invalidations that don't represent changes to the value.  So to do this, we introduce an `oldValue` field, and compare the current value to it (and potentially update it) in `computeValue()`.  The `counter` field keeps track of how many times the value has changed.  

We can use it like this:

``` kotlin
class ComboBoxTest6 : Application() {
    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 400.0, 400.0)
        stage.show()
    }

    private fun createContent(): Region = VBox(20.0).apply {
        val model = Model6()
        children += Label().apply {
            textProperty().bind(model.testValue.map { "The current value in the ComboBox: $it" })
        }
        children += Label().apply {
            textProperty().bind(CountingBinding(model.testValue).map { "Has changed $it times" })
        }
        children += ComboBox(model.valueList) conditionallyBind model.testValue
        padding = Insets(40.0)
    }
}

class Model6 {
    val testValue = SimpleObjectProperty<String>()
    val valueList: ObservableList<String> =
        FXCollections.observableArrayList("abc", "abj", "def", "hij", "mno", "xyz", "123", "456")
}


fun main() = Application.launch(ComboBoxTest6::class.java)
```
And it does what you would expect it to.  The counter increments every time that `model.testValue` changes.

One thing to note is that `Model6.testValue` is now initialized with a `null` value.  That's because, when it's bound to the `ComboBox` its initial value is set to `null` and if we want to avoid starting off with a non-zero counter we need to avoid the first change from "" to `null`.  

The really interesting thing about this `Binding` is that it doesn't return anything that you could consider a manipulation of the values passed to it via its dependencies.  It's something completely unique.  

Another thing you could do is to expand our class to be more than just a `Binding`.  Let's add a `reset()` function to it:

``` kotlin
class CountingBinding2<T>(private val dependency: ObservableObjectValue<T?>) : IntegerBinding() {

    init {
        super.bind(dependency)
    }

    fun reset() {
        counter = 0
        invalidate()
    }

    private var oldValue: T? = dependency.value
    private var counter = 0

    override fun computeValue(): Int {
        if (oldValue != dependency.value) {
            oldValue = dependency.value
            counter++
        }
        return counter
    }
}
```
The important thing here is to remember that this is still essentially a `Binding` and we want it to behave properly.  So the `reset()` function needs to call `invalidate()` which will force any bound elements to call `computeValue()`.

Now we can do this:

``` kotlin
class ComboBoxTest7 : Application() {
    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 400.0, 400.0)
        stage.show()
    }

    private fun createContent(): Region = VBox(20.0).apply {
        val model = Model7()
        val countingBinding = CountingBinding2(model.testValue)
        children += Label().apply {
            textProperty().bind(model.testValue.map { "The current value in the ComboBox: $it" })
        }
        children += Label().apply {
            textProperty().bind(countingBinding.map { "Has changed $it times" })
        }
        children += Button("Reset").apply {
            setOnAction { countingBinding.reset() }
        }
        children += ComboBox(model.valueList) conditionallyBind model.testValue
        padding = Insets(40.0)
    }
}

class Model7 {
    val testValue = SimpleObjectProperty<String>()
    val valueList: ObservableList<String> =
        FXCollections.observableArrayList("abc", "abj", "def", "hij", "mno", "xyz", "123", "456")
}


fun main() = Application.launch(ComboBoxTest7::class.java)
```
Here we've instantiated `countingBinding` as a `CountingBinding2` so that we hava a reference to it that includes access to the `reset()` function.  Then we've added a `Button` that will call `countingBinding.reset()` when it's clicked.  The result looks like this:

{% include video id="EMMHsce1LGA" provider="youtube" %}

This is cool, and probably not something you've thought to do via a `Binding` before.  I know that I hadn't.  

But is this something that you *want* or *should* do?

That's for you to decide.  The key question to ask is whether or not using a technique like this makes your code easier to understand and maintain or not, and there's a lot of other considerations that come into that.  It seems to me that if you're doing this as a "one off", then the uniqueness of this approach is going to require research for a maintenance programmer to understand.  You'd need to offset that against the simplification of the layout code that results from the encapsulation of the functionality.  

On the other hand, if you find yourself using this technique so much that you put the custom `Binding` class in a library and use it over and over, then it's probably worthwhile.

# Conclusion

I think that the idea of a `ConditionalBinding` keeps to the idea of a `Binding` as the connective tissue that holds a Reactive JavaFX design together.  Used as described here, it provides a missing element that makes `ComboBox` work much closer to the way that you (well, at least that "I") would expect it to work.  At the same time, it sticks to idea of a `Binding` as an extension of `ObservableValue` and you can treate `ConditionalBinding` just like any other `ObservableValue` and it will work just fine.

The addition of internal state to a `Binding` is a powerful idea that can greatly expand the utility of `Bindings`, primarily by allowing you to encapsulate and isolate more complicated relationships between data elements.  

If you look at the example code that utilizes `ConditionalBinding`, especially the version that uses the Kotlin extension functions on `ComboBox`, you'll see that the mechanics of dealing with whether or not the pop-up is hidden or visible, and how that impacts updates to the Model, are completely absent from the layout code.  That's the power of encapsulation.  
