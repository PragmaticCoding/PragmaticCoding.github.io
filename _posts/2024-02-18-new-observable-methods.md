---
title:  "New Methods For Observables"
date:   2024-02-18 12:00:00 -0500
categories: java
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/subscribe_and_map
ScreenSnap1: /assets/elements/Subscriptions1.png
ScreenSnap2: /assets/elements/Subscriptions2.png
ScreenSnap3: /assets/elements/Subscriptions3.png
ScreenSnap4: /assets/elements/Subscriptions4.png
ScreenSnap5: /assets/elements/Subscriptions5.png
ScreenSnap6: /assets/elements/Subscriptions6.png
ScreenSnap7: /assets/elements/Subscriptions7.png
ScreenSnap8: /assets/elements/Subscriptions8.png
ScreenSnap9: /assets/elements/Subscriptions9.png
ScreenSnap10: /assets/elements/Subscriptions10.png
ScreenSnap11: /assets/elements/Subscriptions11.png
ScreenSnap12: /assets/elements/Subscriptions12.png
ScreenSnap13: /assets/elements/Subscriptions13.png
excerpt: In JavaFX 19 and 21, new methods have been added to the Observables library that should change the way you write Bindings and Listeners
---

# Introduction

I was working on a JFX 21 project last week and IntelliJ came up with `ObservableValue.map()` as a suggestion.  I looked it up in the JavaDocs, I tried it, it worked, and I moved on with what I was doing.  

For the next few days, however, some thoughts gnawed at my brain: "How had I missed this?", and "Isn't this is a game-changer?".

How *had* I missed this?

I went back and looked, and right at the bottom of the entry... "Since 19".

Then I looked at the JavaDocs page for `ObservableValue` in both JFX 16 and 21.  There are just 3 methods in JFX 16: `addListener()`, `getValue()`, and `removeListener()`.  There are 9 methods in the JFX 21 version.  Those 6 new methods (plus one in `Observable`) are what we are going to look at in this article.

And yes, these are game changers!

# ObservableValue.map() and ObservableValue.orElse()

Let's start by looking at the JavaDocs for [ObservableValue.map()](https://openjfx.io/javadoc/21/javafx.base/javafx/beans/value/ObservableValue.html#map(java.util.function.Function)):

> Returns an ObservableValue that holds the result of applying the given mapping function on this value. The result is updated when this ObservableValue changes. If this value is null, no mapping is applied and the resulting value is also null.

**Hang on!**  This pretty much describes, in a general way, what **every** function in the `Bindings` library does - at least when they are dependant on only one `Observable`.

Let's look at an example.  First, the old `Bindings` library way:

``` kotlin
region.minWidthProperty().bind(Bindings.createDoubleBinding({if (someBooleanProperty.get()) 50.0 else 75.0}, someBooleanProperty))
```
Here, we are just using a `BooleanProperty` to control the `minWidth` of some region in our layout.  If the `BooleanProperty` holds `true` then we'll set the `minWidth` to 50.0 otherwise it will be 75.0.  Whenever the value in `someBooleanProperty` changes, then `region.minWidthProperty()` will change in lock-step.

Now, with `ObservableValue.map()`:

``` kotlin
region.minWidthProperty().bind(someBooleanProperty.map{if (it) 50.0 else 75.0})
```
This does exactly the same thing.  Note that `ObservableValue.map()` returns an `ObservableValue` wrapping the type defined by the `Function` passed to it.  However, just like with the `Bindings` library methods, that `Function` deals with the value inside the `ObservableValue` and returns a non-observable type.  In this example, `it` is `Boolean` and the `Function` returns `Double`.  The function is `Function<Boolean, Double>`.

Clearly this new method is less code, and easier to understand.  It's also easy to view it as an extension of the Fluent API.  You can also chain `Observable.map()` calls together if it makes it more clear.

Another benefit is that it's the same function and format for every kind of conversion that you might want to do.  Instead of having to pick the correct method from `Bindings`, just use map and let the compiler infer the types and off you go.

There's one hitch...

Remember this from the JavaDocs, quoted above?

> If this value is null, no mapping is applied and the resulting value is also null.

Uh oh!  Any `ObservableValue` can potentially hold a `Null` value, in which case we'll get a `Null` back out if it.  In our example, do we really want `Null` if `someBooleanProperty` holds a `Null` value?  Probably not.  

Here comes `orElse()` to the rescue:

``` kotlin
region.minWidthProperty().bind(someBooleanProperty.map{if (it) 50.0 else 75.0}.orElse(60.0))
```
Once again, this will return an `ObservableValue` of the right type, but the parameter passed to `orElse()` is a value of that type, not an `ObservableValue`.

You'll need to keep in mind that since `ObservableValue.map()` returns an `ObservableValue` and not a `Property` of some kind, it cannot be used for a bidirectional binding.

# ObservableValue.flatMap()

My first reaction when seeing this was, "Ho hum, just another `map` function for completeness."  

But then I read the example in the JavaDocs, which starts with this:

```java
ObservableValue<Boolean> isShowing = listView.sceneProperty()
     .flatMap(Scene::windowProperty)
     .flatMap(Window::showingProperty)
     .orElse(false);
```
and the explanation:

> Changes in any of the values of: the scene of listView, the window of that scene, or the showing of that window, will update the boolean value isShowing.

This is big!  

One of the biggest headaches in JavaFX is handling nested `Properties`, or `Properties` of `Properties`.  This pops most often when you're dealing with `TableView` and `ListView`, or really any other container based on `VirtualFlow`.  That's because the cells have a `Property` called `item` and it contains the value currently held in the cell.  When that value is a simple object or primitive, like a `String` then it's easy simply bind some `Node` in the cell layout to `item` and it will work.

But when `item` is an object composed of other `Observables` then it becomes harder to do this.  Binding, say, `label.textProperty()` to `item.nameProperty()` is problematic because of the reach-through to the component field.  When `item` changes, the binding goes with it, and now `label.textProperty()` is bound to the `nameProperty()` of some prior value of `item` and not the current one.  

This was particularly irksome with `ListView` where the `items` passed to the `ListCells` are often complex objects composed of `Observable` fields.

There was a way around this, and that was `Bindings.select()`.  It was problematic, though, because it used reflection and depended on the names used for the component fields following a pattern.  

`ObservableValue.flatMap()` fixes that.  As a matter of fact, the current JavaDocs for `Bindings.select()` tell you to use `Observable.flatMap()`.  This is somewhat analogous to how `PropertyValueFactory` should be replaced with lambda calls after Java 8.  

## ObservableValue.flatMap() Example

Let's take a look at how this works.  One of the most common uses for nested `Properties` is to display information from the selected item in a `TableView`.  This is usually done by linking through the `SelectedItemProperty` of a `TableView`, which, in turn, is normally composed of other `Properties`.  

Here's a simple example of how this works with `ObservableValue.flatMap()`

``` kotlin
class FlatMap : Application() {

    val selectedItem: ObjectProperty<TableData> = SimpleObjectProperty()

    override fun start(stage: Stage) {
        val scene = Scene(createContent(), 620.0, 300.0).apply {
            addStyleSheet("/ca/pragmaticcoding/widgetsfx/css/LabelBox.css")
            addWidgetStyles()
        }
        stage.title = "LabelBox Demo"
        stage.scene = scene
        stage.show()
    }

    fun createContent() = BorderPane().apply {
        padding = Insets(20.0, 20.0, 20.0, 20.0)
        styleClass += "wrapper-region"
        center = createTable()
        right = createPane()
    }

    private fun createTable(): Region = TableView<TableData>().apply {
        columns += TableColumn<TableData, String>("Name").apply {
            cellValueFactory = Callback { it.value.nameProperty }
        }
        columns += TableColumn<TableData, String>("Address").apply {
            cellValueFactory = Callback { it.value.addressProperty }
        }
        items += listOf(
            TableData("Wizard of Oz", "1 Yellow Brick Road"),
            TableData("Herman Munster", "1313 Mockingbird Lane"),
            TableData("Norman Bates", "Bates Motel")
        )
        selectedItem.bind(selectionModel.selectedItemProperty())
        columnResizePolicy = TableView.CONSTRAINED_RESIZE_POLICY_LAST_COLUMN
        maxWidth = 300.0
    }

    private fun createPane(): Region = VBox(10.0).apply {
        children += HBox(10.0, Label("Name: "), Label().apply {
            textProperty().bind(selectedItem.flatMap { it.nameProperty })
        })
        children += HBox(10.0, Label("Address: "), Label().apply {
            textProperty().bind(selectedItem.flatMap { it.addressProperty })
        })
        minWidth = 300.0
        padding = Insets(15.0)
    }
}

class TableData(val name: String, val address: String) {
    val nameProperty: StringProperty = SimpleStringProperty(name)
    val addressProperty: StringProperty = SimpleStringProperty(address)
}


fun main() {
    Application.launch(MapsAndSubscriptions::class.java)
}
```
Our `Model` for the `TableView` is `TableData`, which is a object composed of two `StringPropertie`, `nameProperty` and `addressProperty`.  The `TableView` just has two columns, one for "Name" and one for "Address".  Off to the right of the `TableView` we have a `VBox` with two `Label` pairs in `HBoxes`.  One for the name, and one for the address.

There's an `ObjectProperty<TableData>` field in our application.  It's bound to the `TableView's selectedItemProperty`, so it will always reflect the currently selected item in the `TableView`.

Two of the `Labels` on the right side need to bound to the corresponding `Properties` in value held in `selectedItem`, and this is done via `flatMap()`:

``` kotlin
  textProperty().bind(selectedItem.flatMap { it.nameProperty })
```
and:
``` kotlin
  textProperty().bind(selectedItem.flatMap { it.addressProperty })
```
Now, when either `selectedItem` changes, or either of the two `Properties` that make up that current value changes, the `Labels` will also change.

When we run this, we first see:

![FlatMap 1]({{page.ScreenSnap8}})

Then, when we select an item:

![FlatMap 2]({{page.ScreenSnap9}})

And, then another:

![FlatMap 3]({{page.ScreenSnap10}})

The one thing that you cannot do with this is to change those data `Labels` to `TextField` and bidirectionally bind their `TextProperty` to the `flatMap()` value, because it returns `ObservableValue` and not a `Property` of some sort.

# ObservableValue.subscribe() and Observable.subscribe()

`Subscription` is another neat new feature, this one added in JFX 21.  From the description in the [issue](https://bugs.openjdk.org/browse/JDK-8304439), I can paraphrase the situation that they were trying to solve:

> `Listeners` are heavyweight and contain features that application programmers seldom use.  `ChangeListener` is often used when `InvalidationListener` would suffice, and the references to `ObservableValue` and the `oldValue` are seldom seen in application code.  Consequently, `Listeners` are cumbersome to implement in application code.  The inclusion of `Lambdas` and `Method references` to the language make it increasingly difficult to remove `Listeners` from `Observables`.

This has been my observation when looking at other programmers' projects also.  `ChangeListener` is used almost exclusively, even though `InvalidationListener` is more appropriate in many cases.  At times, the `Listener` is only used to trigger something which doesn't even need the new value of the `Observable`.  

Personally, I now rarely work on the kind of projects where garbage collection is an issue, and defunct `Listener` references to `Observables` prevent those `Observables` from being garbage collected, but I can see the need to be able to remove `Listeners` without bloating the code base with boilerplate to achieve this.

## The Differences between InvalidationListener and ChangeListener

Before we go on, it's best to have a quick review about how the two `Listener` types in JavaFX work.

The first concept to understand is "invalidation".  An `Observable` is considered to have a "valid" value when it's, well, valid - or known to be correct.  If an `Observable` is bound to another `Observable` in some way, and that other `Observable` changes, then it's *possible* that the value of the bound `Observable` has also changed.  Under the hood, whenever an `Observable` is changed, JavaFX gets a list of all the other `Observables` that are bound to it and changes their status from "valid", to "invalid" (a process called "invalidation").  Additionally, when an `Observable` is invalidated, JavaFX gets a list of all of the other `Observables` that are bound to it and invalidates them, too.

Finally, `Observables` are flipped back to "valid" when `get()` or `getValue()` are called on them.

An `InvalidationListener` triggers when an `Observable` has been invalidated.  It doesn't mean that the value *has* changed, just that it *might have* changed.  Consider the following code:

``` kotlin
val a : IntegerProperty = SimpleIntegerProperty(10)
val b : BooleanProperty = a.map{if (it > 20) true else false}
b.addListener(InvalidationListener{println("I was triggered ${b.value}")})
a.value = a.value + 1
a.value = a.value + 20
```
After the second line, the value of `a` was 10 and `b` was false, `b` was valid.  After the first increment of `a`, the value of `a` was 11, and b was invalidated.  This would trigger the `Listener` and, "I was triggered false" would print.  After this printed, `b` would be valid again. After the last line `a` was 31 and b was invalidated again.  This would trigger the `Listener` and "I was triggered true" would print. And then, once again, `b` would be valid.

However, if we take out the `${b.value}` from the `println`, so that the code looks like this:
``` kotlin
val a : IntegerProperty = SimpleIntegerProperty(10)
val b : BooleanProperty = a.map{if (it > 20) true else false}
b.addListener(InvalidationListener{println("I was triggered")})
a.value = a.value + 1
a.value = a.value + 20
```
Things are different.

After the second line, the value of `a` was 10 and `b` was false, `b` was valid.  After the first increment of `a`, the value of `a` was 11, and b was invalidated.  This would trigger the `Listener` and, "I was triggered" would print.  After this printed, `b` would still be invalid. After the last line `a` was 31 and `b` was already invalid.  This would **not** trigger the `Listener` and nothing would print.  And then, at the end, `b` would still be invalid.

The best mental model of a `ChangeListener` is to think of it as a wrapper around an `InvalidationListener`, with a place to hold the last value stored in the `Observable`.  When the `InvalidationListener` triggers, it invokes the `Observable.get()` method, revalidating the `Observable`, and comparing the results to the previous value that it has stored.  If these two values are different, then it triggers the `ChangeListener` action.

The key differences between `InvalidationListener` and `ChangeListener` are that `ChangeListener` only triggers the action on an actual change of the value, and it always re-validates the `Observable` automatically, even if the invalidation didn't result in a value change.  Beyond that, `ChangeListener` gives you old and new values, if your logic needs to compare them.  

With that in mind, let's look at the new subscription facility in JavaFX...

## Using Subscriptions

This feature introduces a new `Interface` called `Subscription` which allows you to track and combine the subscriptions you've added to your `Observables`.  `Subscriptions` are returned from the `subscribe()` methods.  The `Subscription` interface has methods that allow you to combine `Subscriptions` and to unsubscribe `Subscriptions`.  

All of that sounds very circular.  `Subscription` doesn't really talk about what they do, just how to combine them and get rid of them.  The idea is that you can create a compound `Subscription` that acts as a container for all of your individual `Subscriptions` so that when you are done with them you can unsubscribe them all at once.  And it's all easy, easy, easy to do.

There are three `subscription()` methods; two in `ObservableValue` and one in `Observable`.  They differ in only the argument that they take.  Here's what they do when triggered:

1. ObservableValue.subscribe(Consumer)<br>Will invoke a `Consumer` that accepts the new value of the `ObservableValue`.
1. ObservableValue.subscribe(BiConsumer)<br>Will invoke a `BiConsumer` that accepts the old value and the new value of the `ObservableValue`.
1. Observable.subscribe(Runnable)<br>Will invoke a `Runnable` that does accept not any values.

You have to read the JavaDocs fairly carefully to get all the nuances in here.  Let's experiment...

### Using Observable.subscribe()

Let's look at the first version, which just takes a runnable as its argument.  How does this interact with the Invalidation status of the `Observable`.

Here's some test code:

``` kotlin
class MapsAndSubscriptions : Application() {
    override fun start(stage: Stage) {
        val scene = Scene(createContent(), 420.0, 300.0).apply {
            addStyleSheet("/ca/pragmaticcoding/widgetsfx/css/LabelBox.css")
            addWidgetStyles()
        }
        stage.title = "LabelBox Demo"
        stage.scene = scene
        stage.show()
    }

    fun createContent() = BorderPane().apply {
        padding = Insets(20.0, 20.0, 20.0, 20.0)
        styleClass += "wrapper-region"
        center = VBox(20.0, createPane())
    }

    private fun createPane(): Region = VBox(10.0).apply {
        val counter: IntegerProperty = SimpleIntegerProperty(0)
        val textArea = TextArea()
        counter.addListener(InvalidationListener { textArea.text += "Invalidation Listener triggered\n" })
        counter.subscribe(Runnable { textArea.text += "Subscription triggered\n" })
        children += Button("Click Me").apply {
            val buttonCounter: IntegerProperty = SimpleIntegerProperty(0)
            textProperty().bind(buttonCounter.map { "Click Me $it" })
            onAction = EventHandler {
                textArea.text += "Button clicked ${buttonCounter.value}\n"
                counter.value = buttonCounter.value++
            }
        }
        children += textArea
    }
}


fun main() {
    Application.launch(MapsAndSubscriptions::class.java)
}
```
This is going to give us a window with a `Button` and a `TextArea`.  Every time the `Button` is clicked, it's going to add some text to the `TextArea` and update an `IntegerProperty` called `counter`.  In addition, it will increment another `IntegerProperty` called `buttonCounter` which is displayed on the `Button` through a `map()`.  

Additionally, there is an `InvalidationListener` on `counter` that does **not** revalidate `counter` by calling its `get()` method.  Nor does the `Subscription`.

Let's see what happens when you click the `Button` a few times:

![Screen Snap 1]({{page.ScreenSnap1}})

On the first click, `buttonCounter` is 0 and it sets `counter` to 0, which is what it already was.  So this does not invalidate `counter`.

On the second click, `buttonCounter` is 1 and it sets `counter` to 1, which then invalidates `counter`.  Both the `InvalidationListener` and the `Subscription` are triggered.  

On the subsequent clicks, `counter` is incremented, but it's already been invalidated so neither the `InvalidationListener` nor the `Subscription` are triggered again.

Now, let's change one line of code; the line that creates the `InvalidationListener`:

``` kotlin
counter.addListener(InvalidationListener { textArea.text += "Invalidation Listener triggered ${counter.value}\n" })
```
This calls `counter.get()` through the Kotlin property accessor `value`.  So this **will** revalidate `counter`.  Here's the output:

![Screen Snap 2]({{page.ScreenSnap2}})

Finally, we put the `InvalidationListener` code back the way it started, and change the `Subscription` line so that it will revalidate `counter`:

``` kotlin
counter.subscribe(Runnable { textArea.text += "Subscription triggered ${counter.value}\n" })
```
![Screen Snap 3]({{page.ScreenSnap3}})

From this, I think it's safe to assume that `Observable.subscribe()` is analogous to adding an `InvalidationListener` to your `Observable`.  This is in agreement with the JavaDocs, which state:

> The provided subscriber is akin to an InvalidationListener without the Observable parameter.

### ObservableValue.subscribe()

Now let's look at the two methods defined in `ObservableValue`.  These are much more like `ChangeListener`.

#### With a Consumer

The next version of `subcribe()` that we'll look at is the one that takes a `Consumer`.  We'll leave all the rest of the example code alone, including the `InvalidationListener,` but change the subscription:

``` kotlin
counter.subscribe(Consumer { textArea.text += "Subscription triggered \n" })
```
Note that we're not accessing the value passed to the `Consumer` here.  This is what we get:

![Screen Snap 4]({{page.ScreenSnap4}})

This is *almost* what you'd expect. Nothing happens after the first `Button` click because the value doesn't change, it stays 0.  After that, the `Subscription`, in getting the value to pass to the `Consumer` is revalidating our `Property`.  So we expect to see both the `Subscription` and the `InvalidationListener` trigger on each subsequent `Button` click.

But the very first "Subscription Triggered" message is interesting.  It clearly happens when the `Subscription` is created.  Let's check the JavaDocs on that:

> Creates a Subscription on this ObservableValue which immediately provides the current value to the given valueSubscriber, followed by any subsequent values whenever its value changes. The valueSubscriber is called immediately for convenience, since usually the user will want to initialize a value and then update on changes.

Ah!  When I first read this it wasn't clear to me what "immediately provides" meant.  Now it's clear, as is the explanation about initializing a value as a use case.

Aside from this, it's safe to consider this version of `subscribe()` to be about the same as a `ChangeListener` that only passes the `newVal` to its `Handler`.

#### With a BiConsumer

Now we'll swap out the `Subscription` to use the last version, the one that passes both the old and the new values to a `BiConsumer`:

``` kotlin
counter.subscribe { _: Number, _: Number -> textArea.text += "Subscription triggered \n" }
```
This is the Kotlin version of a BiConsumer.  We're not going to use the values passed in, so `_` is used a a placeholder.

The output looks like this:
![Screen Snap 5]({{page.ScreenSnap5}})

This is also pretty much what you'd expect, except that we **don't** get the initial firing of the `Subscription` when it's created.  Check the JavaDocs and you'll see that there is no mention of "immediately provides" for this version of `subscribe()`.

#### Invalidation Without a Value Change

We know from the discussion above, that an `Observable` can invalidate because something that it's bound to has been invalidated, while its own value doesn't actually change.  Let's see how that works with `Subscriptions`.  We're going to add a new `Observable` to our code, that is bound to `counter` through a map that means it won`t always change when `counter` changes.

``` kotlin
val counter: IntegerProperty = SimpleIntegerProperty(0)
val boundCounter = counter.map { if ((it as Int) < 3) 10 else 20 }
boundCounter.addListener(InvalidationListener { textArea.text += "Invalidation Listener triggered\n" })
boundCounter.subscribe { _: Number, _: Number -> textArea.text += "Subscription triggered \n" }
```
All the rest of the code is the same.  Here's the output:
![Screen Snap 6]({{page.ScreenSnap6}})

This is what I would have expected (hoped?) would happen.  For the second and third button clicks, `counter` changes and invalidates which, in turn, invalidates `boundCounter`.  At the same time, the `Subscription`, which **doesn't** trigger the BiConsumer, *does* re-validate the `Observables` because it needs to call `get()` on `boundCounter` in order to decide if it has changed.  

We'll try the same thing, but with the `Consumer` version:
![Screen Snap 7]({{page.ScreenSnap7}})

This is exactly the same, except that we have that initial triggering when the `Subscription` is created.  

#### Which to Use?

I think that for 95% of use cases - which is when you really want to trigger when the value actually changes, but you don't care what the previous value was - then you should just use the `ObservableValue.subscribe(Consumer)` version.  Use this even if you don't want to use the new value in anything directly.  This is going to free you up from having to worry about invalidation issues.  On those few cases that you do need the previous value, then use the `BiConsumer` version.

I would reserve the `Runnable` version for those rare cases when you really do care about the difference between invalidation and change, and you want to fire whenever anything in your chain of bound `Properties` changes.

The other big thing to think about is this initial firing of the `Consumer` version when it's initiated.  I've seen applications where a `Property` is used to connect to modules of a together, and it's passed from the first module to the second via a constructor parameter.  Then the dependent module uses a `ChangeListener` to do something when the controlling module changes the value of the `Property`.  In that situation, you usually want to run whatever code executes when the value changes when you first get the property, because often it already has a valid value in it.  So you're forced to put the `ChangeListener` code into a method so that you can manually force it to run from the constructor.  With this version of `Subscription`, you don't need to do that.

## Combining and Unsubscribing

One of the big advantages to `Subscriptions` is that it's easier to remove them than the corresponding `Listener` type, especially when the `Listener` was declared via a lambda or method reference.  This is because the `removeListener()` functions take a reference to `Listener` to be removed.  For instance, in our example code above, we have this:

``` kotlin
boundCounter.addListener(InvalidationListener { textArea.text += "Invalidation Listener triggered\n" })
```
If we want to be able to remove this listener, then we have to do this:
``` kotlin
val listener = InvalidationListener { textArea.text += "Invalidation Listener triggered\n" }
boundCounter.addListener(listener)
```
and then, later on we can do:
``` kotlin
boundCounter.removeListener(listener)
```
While `addListener()` has a return type of `void`, `subscribe()` has a return type of `Subscription`.  And the `Subscription` interface has a method called `unsubscribe`.  Back to our example code, we can do this:

``` kotlin
val subscription = boundCounter.subscribe { _: Number, _: Number -> textArea.text += "Subscription triggered \n" }
```
Which means that later we can do this:

``` kotlin
subscription.unsubscribe()
```
Now, if you have a number of `Subscriptions` and you don't want to keep track of them individually throughout your code, you can do something like this:

``` kotlin
val subscriptions = Subscription.EMPTY
   .
   .
subscriptions.add(boundCounter.subscribe { _: Number, _: Number -> textArea.text += "Subscription triggered \n" })
   .
   .
subscriptions.unsubscribe()
```
alternatively, you can do something like this:

``` kotlin
val subscriptions = Subscription.combine(subscription1, subscription2, subscription3)
   .
   .
subscriptions.unsubscribe()
```


# ObservableValue.when()

`Observable.when()` is described primarily as a means to avoid having defunct references which prevent garbage collection of otherwise unused `Properties`.  But how does it work, and what can we expect to see when we use `when()`

## ObservableValue.when() Example

``` kotlin
class WhenApplication : Application() {
    override fun start(stage: Stage) {
        val scene = Scene(createContent(), 420.0, 300.0).apply {
            addStyleSheet("/ca/pragmaticcoding/widgetsfx/css/LabelBox.css")
            addWidgetStyles()
        }
        stage.title = "LabelBox Demo"
        stage.scene = scene
        stage.show()
    }

    fun createContent() = BorderPane().apply {
        padding = Insets(20.0, 20.0, 20.0, 20.0)
        styleClass += "wrapper-region"
        center = VBox(20.0, createPane())
    }

    private fun createPane(): Region = VBox(10.0).apply {
        val textArea = TextArea()
        val bindingEnabled: BooleanProperty = SimpleBooleanProperty(false)
        val counter: IntegerProperty = SimpleIntegerProperty(0)
        val boundCounter: IntegerProperty = SimpleIntegerProperty(25)
        boundCounter.bind(counter.`when`(bindingEnabled).map { it.toInt() + 17 })
        boundCounter.subscribe { it: Number -> textArea.text += "Subscription triggered $it\n" }
        bindingEnabled.subscribe { it: Boolean -> textArea.text += "Binding enabled: $it\n" }
        children += Button("Click Me").apply {
            val buttonCounter: IntegerProperty = SimpleIntegerProperty(0)
            textProperty().bind(buttonCounter.map { "Click Me $it" })
            onAction = EventHandler {
                textArea.text += "Button clicked ${buttonCounter.value}\n"
                counter.value = buttonCounter.value++
            }
        }
        children += CheckBox("Binding On").apply {
            selectedProperty().bindBidirectional(bindingEnabled)
        }
        children += HBox(10.0, Label("Bound Counter:"), Label().apply { textProperty().bind(boundCounter.asString()) })
        children += textArea
    }
}
```
A quick Kotlin note: `when` is a reserved keyword in Kotlin, so to use `when()` we need put the "when" in backticks.

Here we have a couple if `IntegerProperties`, one called `boundCounter` which is bound to the other, called `counter` through a `map()` and a `when()`.  The `map()` just adds 17 to the value in `counter`.  There is a `BooleanProperty` called `bindingEnabled` which is bound bidirectionally to a `CheckBox`.  

With this we can control the `when()` with the `CheckBox` and increment `counter` via the `Button`.  Then we can see how it works.

Let's look at the output from this.  First, once the application has been started and the Button has been clicked twice:

![When Snap 1]({{page.ScreenSnap11}})

We see the output from creating the two `Subscriptions`, as they fire immediately.  Interestingly, even though the value in `bindingEnabled` is `false`, and `boundCounter` was initialised to 25, it's shows as 17 when the `Subscription` was added.  There's evidence of two `Button` clicks, and `counter` now has a value of 1 (remember, it's one behind the number shown on the `Button`).  Yet the value in `boundCounter` remains at 17.  

Next, we'll click on the `CheckBox` to active the `when()` and click the `Button` again.  After that, we'll unselect the `CheckBox`:

![When Snap 2]({{page.ScreenSnap12}})

We see the output from the two subscriptions when the `when()` is activated.  One is for `bindingEnabled` changing to `true`, and the other is when `boundCounter` is updated because the `when()` is activated.  Then the `Button` is clicked, and we see the subscription on `boundCounter` firing, telling us the value in now 19.  Finally, we see the `bindingEnabled` subscription firing when the `CheckBox` was unselected.  The current value of `boundCounter` remains 19.

Finally, we'll click the `Button` again:

![When Snap 3]({{page.ScreenSnap13}})

Here we see `Button` click message, but nothing else.  The value of `boundCounter` remains 19.


# Conclusion

In my opinion, `ObservableValue.map()`, and the `Subscription` methods are "game changers" for writing Reactive JavaFX applications.  Using these methods will result in cleaner, simpler code with less boilerplate.  By themselves, these are reason enough to start using JFX 21 right now.
