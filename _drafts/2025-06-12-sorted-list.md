---
title:  "ObservableLists: FilteredList & SortedList"
date:   2025-06-12 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/filtered-sorted-lists
PantsOnFire: /assets/images/PantsOnFire.png
ScreenSnap0: /assets/elements/TransformList0.png
ScreenSnap1: /assets/elements/TransformList1.png
ScreenSnap2: /assets/elements/TransformList2.png
ScreenSnap3: /assets/elements/TransformList3.png
ScreenSnap4: /assets/elements/TransformList4.png
ScreenSnap5: /assets/elements/TransformList5.png
ScreenSnap6: /assets/elements/TransformList6.png
ScreenSnap7: /assets/elements/TransformList7.png
ScreenSnap8: /assets/elements/TransformList8.png
ScreenSnap9: /assets/elements/TransformList9.png
ScreenSnap10: /assets/elements/TransformList10.png
ScreenSnap11: /assets/elements/FilterList1.gif

Diagram: /assets/elements/ListProperties.png
OLArticle: /javafx/elements/observable-classes-lists
OBGuide: /javafx/elements/observables_guide

JavaDocOL: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/ObservableList.html
JavaDocFXC: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/FXCollections.html

excerpt: "Explaining how to create and use two subclasses of ObservableList: FilteredListed and SortedList"
---

# Introduction

Both `FilteredList` and `SortedList` are "wrapper" classes that enclose an instance of `ObservableList` and provide additional functionality to that `ObseravbleList`.  

`FilteredList`
: Allows you to add a `Predicate` to an `ObservableList`, which will cause only certain items to be seen.  

`SortedList`
: Allows you to add a `Comparator` to an `ObservableList`, which will determine the order in which the items will be seen.

The thing that makes these two classes different from just sorting a `List` or extracting the elements that you want into a second `List` is that they are "live".  The original `List`, being an `ObservableList` continues to be watched, and changes in that `ObservableList` will be reflected in the `FilteredList` and the `SortedList`.

# The Basics

We are going to look at `FilteredList` and `SortedList` together because they share a great deal of their underlying concepts.

## TransformationList

The first of these is that they are both subclasses of the abstract class `TransformationList`.  Let's take a quick look at `TransformationList` so that we can understand these two implementations of it.

`TransformationList` extends from `ObservableListBase` which implements `ObservableList`.  So `TransformationList` and its subclasses will also implement `ObservableList`.

`TransformationList` has a public method called `getSource()` which allows you to get a reference to the enclosed `ObservableList`.  It also has two abstract public methods called `getViewIndex()` and `getSourceIndex()` which allow you to map item references between the `TransformationList` and its enclosed `ObservableList`.

Finally, we have the most interesting method, it's a protected abstract method called `sourceChanged(ListChangeListener.Change<? extends F> c)`.  Let's ignore the generic type stuff for now, and look at what this means.  It accepts a `ListChangeListener.Change` parameter.  So we now know that the `TransformationList` is kept synchronized with the underlying `ObservableList` by processing the individual changes to that `ObservableList`.  This might be useful in understanding how to manage `SortedList` and `FilteredList`.

## Creating a FilteredList or SortedList

There are two methods that you can use instantiate either of these `Lists`.  The first is through constructors for the `Lists`.  Each class has two constructors, one that simply takes an `ObservableList`, and one that takes a second parameter that controls the filtering or sorting:

``` kotlin
val obList = FXCollections.observableArrayList<Int>()
val filteredList1 = FilteredList(obList)
val filteredList2 = FilteredList(obList, { item -> item <= 22 })

val sortedList1 = SortedList(obList)
val sortedList2 = SortedList(obList, { a, b -> a.compareTo(b) })
```
The second technique is to use methods in `ObservableList`:

``` kotlin
val obList = FXCollections.observableArrayList<Int>()
val filteredList1 = obList.filtered({ item -> item <= 22 })

val sortedList1 = obList.sorted()
val sortedList2 = obList.sorted({ a, b -> a.compareTo(b) })
```
Note that `ObservableList.sorted()` with no `Comparator` specified will create a `SortedList` sorted in the "natural" order for whatever type is in the `ObservableList`.

## FilteredList

We'll look at `FilteredList` first, and use the same kind of demo application that I've used for other `ObservableList` articles.  This is a screen with a `TextArea` bound to a `StringProperty` called `messages`, and then we'll use `EventHandlers` and `Subscriptions` to update `messages` as things happen.  There's also a couple of `Buttons` that will manipulate our `ObservableList`.   

Additionally we'll have two `ListViews`, one for the `ObservableList` and one for the `FilteredList` based on it.  

{% include notice_kotlin %}

Here's the code:

``` kotlin
class FilterListExample1 : Application() {

    private val obList: ObservableList<Int> = FXCollections.observableArrayList()
    private val filterList = FilteredList(obList) { item -> item > 6 }
    private val messages: StringProperty = SimpleStringProperty("")
    private val obTotalBinding = TotalBinding(obList)
    private val filterTotalBinding = TotalBinding(filterList)

    override fun start(stage: Stage) {
        populateList(obList)
        obList.subscribe { messages.value += "obList Invalidated \n" }
        filterList.subscribe { messages.value += "filterList Invalidated \n" }
        stage.scene = Scene(createContent()).addWidgetStyles()
        stage.show()
    }

    private fun createContent(): Region = BorderPane().apply {
        center = TextArea().apply {
            textProperty().bind(messages)
        }
        left = HBox(
            10.0,
            VBox(
                6.0,
                HBox(
                    20.0, promptOf("obList"),
                    Label().bindTo(obTotalBinding.asString())
                ) alignTo Pos.CENTER_LEFT,
                ListView<Int>(obList).apply { maxWidth = 100.0 }),
            VBox(
                6.0,
                HBox(
                    10.0, promptOf("filterList"),
                    Label().bindTo(filterTotalBinding.asString())
                ) alignTo Pos.CENTER_LEFT,
                ListView<Int>(filterList).apply { maxWidth = 100.0 })
        )
        bottom = HBox(
            20.0,
            Button("Add 3").apply {
                setOnAction {
                    messages.value += "\n3 Button Clicked\n"
                    obList.add(3)
                }
            },
            Button("Add 15").apply {
                setOnAction {
                    messages.value += "\n15 Button Clicked\n"
                    obList.add(15)
                }
            },
        ) padWith 10.0
        padding = Insets(20.0)
    }

    private fun populateList(list: MutableList<Int>) {
        for (x in 0..10) {
            list += x
        }
    }
}

class TotalBinding(private val list: ObservableList<Int>) : IntegerBinding() {
    init {
        super.bind(list)
    }

    override fun computeValue(): Int = list.sum()
}

fun main() = Application.launch(FilterListExample1::class.java)
```
And here's what it looks like:

![Screen Snap]({{page.ScreenSnap0}})

There's a custom `IntegerBinding` called `TotalBinding` that is used to maintain the total amounts shown beside the titles of the two `ListViews`.  There is a `Subscription` on each `List` that displays when they have been invalidated.  

The `FilteredList` is instantiated with this line:

``` kotlin
private val filterList = FilteredList(obList) { item -> item > 6 }
```
You can see that it filters out any value less than or equal to 6.  Comparing the two `ListViews` you can see that this has happened.  The totals are also correct for each `List`.

Here, the "Add 3" `Button` was clicked, then the "Add 15" `Button` two times, and then the "Add 3" `Button` one last time.  You can see the sequence "3,15,15,3" at the bottom of the `obList ListView`, while you can see the sequence "15,15" at the bottom of the `filterList ListView`.  

Notice that all of the additions were done to `obList`, and yet both `Lists` were updated if the `Predicate` was satisfied.

What happens if I change the `Buttons` to update `filterList` instead?  I tried this:

``` kotlin
Button("Add 3").apply {
    setOnAction {
        messages.value += "\n3 Button Clicked\n"
        filterList.add(3)
    }
},
Button("Add 15").apply {
    setOnAction {
        messages.value += "\n15 Button Clicked\n"
        filterList.add(15)
    }
}
```
This is the output:

```
Exception in thread "JavaFX Application Thread" java.lang.UnsupportedOperationException
	at java.base/java.util.AbstractList.add(AbstractList.java:155)
	at java.base/java.util.AbstractList.add(AbstractList.java:113)
	at ca.pragmaticcoding.blog@0.1.0-SNAPSHOT/ca.pragmaticcoding.blog.transformlist.FilterListExample1.createContent$lambda$11$lambda$8$lambda$7(FilterListExample1.kt:66)
	at javafx.base@23/com.sun.javafx.event.CompositeEventHandler.dispatchBubblingEvent(CompositeEventHandler.java:86)
	at javafx.base@23/com.sun.javafx.event.EventHandlerManager.dispatchBubblingEvent(EventHandlerManager.java:232)
	at javafx.base@23/com.sun.javafx.event.EventHandlerManager.dispatchBubblingEvent(EventHandlerManager.java:189)
	at javafx.base@23/com.sun.javafx.event.CompositeEventDispatcher.dispatchBubblingEvent(CompositeEventDispatcher.java:59)
	at javafx.base@23/com.sun.javafx.event.BasicEventDispatcher.dispatchEvent(BasicEventDispatcher.java:58)
	at javafx.base@23/com.sun.javafx.event.EventDispatchChainImpl.dispatchEvent(EventDispatchChainImpl.java:114)
	at javafx.base@23/com.sun.javafx.event.BasicEventDispatcher.dispatchEvent(BasicEventDispatcher.java:56)
	at javafx.base@23/com.sun.javafx.event.EventDispatchChainImpl.dispatchEvent(EventDispatchChainImpl.java:114)
	at javafx.base@23/com.sun.javafx.event.BasicEventDispatcher.dispatchEvent(BasicEventDispatcher.java:56)
	at javafx.base@23/com.sun.javafx.event.EventDispatchChainImpl.dispatchEvent(EventDispatchChainImpl.java:114)
	at javafx.base@23/com.sun.javafx.event.EventUtil.fireEventImpl(EventUtil.java:74)
	at javafx.base@23/com.sun.javafx.event.EventUtil.fireEvent(EventUtil.java:49)
	at javafx.base@23/javafx.event.Event.fireEvent(Event.java:199)
	at javafx.graphics@23/javafx.scene.Node.fireEvent(Node.java:8963)
	at javafx.controls@23/javafx.scene.control.Button.fire(Button.java:203)
	at javafx.controls@23/com.sun.javafx.scene.control.behavior.ButtonBehavior.mouseReleased(ButtonBehavior.java:207)
	at javafx.controls@23/com.sun.javafx.scene.control.inputmap.InputMap.handle(InputMap.java:274)
	at javafx.base@23/com.sun.javafx.event.CompositeEventHandler$NormalEventHandlerRecord.handleBubblingEvent(CompositeEventHandler.java:247)
	at javafx.base@23/com.sun.javafx.event.CompositeEventHandler.dispatchBubblingEvent(CompositeEventHandler.java:80)
	at javafx.base@23/com.sun.javafx.event.EventHandlerManager.dispatchBubblingEvent(EventHandlerManager.java:232)
	at javafx.base@23/com.sun.javafx.event.EventHandlerManager.dispatchBubblingEvent(EventHandlerManager.java:189)
	at javafx.base@23/com.sun.javafx.event.CompositeEventDispatcher.dispatchBubblingEvent(CompositeEventDispatcher.java:59)
	at javafx.base@23/com.sun.javafx.event.BasicEventDispatcher.dispatchEvent(BasicEventDispatcher.java:58)
	at javafx.base@23/com.sun.javafx.event.EventDispatchChainImpl.dispatchEvent(EventDispatchChainImpl.java:114)
	at javafx.base@23/com.sun.javafx.event.BasicEventDispatcher.dispatchEvent(BasicEventDispatcher.java:56)
	at javafx.base@23/com.sun.javafx.event.EventDispatchChainImpl.dispatchEvent(EventDispatchChainImpl.java:114)
	at javafx.base@23/com.sun.javafx.event.BasicEventDispatcher.dispatchEvent(BasicEventDispatcher.java:56)
	at javafx.base@23/com.sun.javafx.event.EventDispatchChainImpl.dispatchEvent(EventDispatchChainImpl.java:114)
	at javafx.base@23/com.sun.javafx.event.EventUtil.fireEventImpl(EventUtil.java:74)
	at javafx.base@23/com.sun.javafx.event.EventUtil.fireEvent(EventUtil.java:54)
	at javafx.base@23/javafx.event.Event.fireEvent(Event.java:199)
	at javafx.graphics@23/javafx.scene.Scene$MouseHandler.process(Scene.java:3987)
	at javafx.graphics@23/javafx.scene.Scene.processMouseEvent(Scene.java:1893)
	at javafx.graphics@23/javafx.scene.Scene$ScenePeerListener.mouseEvent(Scene.java:2711)
	at javafx.graphics@23/com.sun.javafx.tk.quantum.GlassViewEventHandler$MouseEventNotification.run(GlassViewEventHandler.java:411)
	at javafx.graphics@23/com.sun.javafx.tk.quantum.GlassViewEventHandler$MouseEventNotification.run(GlassViewEventHandler.java:301)
	at java.base/java.security.AccessController.doPrivileged(AccessController.java:400)
	at javafx.graphics@23/com.sun.javafx.tk.quantum.GlassViewEventHandler.lambda$handleMouseEvent$2(GlassViewEventHandler.java:450)
	at javafx.graphics@23/com.sun.javafx.tk.quantum.QuantumToolkit.runWithoutRenderLock(QuantumToolkit.java:430)
	at javafx.graphics@23/com.sun.javafx.tk.quantum.GlassViewEventHandler.handleMouseEvent(GlassViewEventHandler.java:449)
	at javafx.graphics@23/com.sun.glass.ui.View.handleMouseEvent(View.java:560)
	at javafx.graphics@23/com.sun.glass.ui.View.notifyMouse(View.java:946)
	at javafx.graphics@23/com.sun.glass.ui.gtk.GtkApplication._runLoop(Native Method)
	at javafx.graphics@23/com.sun.glass.ui.gtk.GtkApplication.lambda$runLoop$10(GtkApplication.java:264)
	at java.base/java.lang.Thread.run(Thread.java:1575)
```
Uh, oh!  

It turns out that the `ObservableList` that `FXCollections.observableArrayList()` returns is a subclass of `ModifiableObservableListBase`.  This class adds contains some methods like `doAdd()` and `doRemove()` that delegate those actions to the backing `ArrayList`.

However, `FilteredList` doesn't extend `ModifiableObservableListBase` and doesn't have any of those methods.  This means that you cannot directly make changes to a `FilteredList`.  To do so, you need to hang on to a reference to the orignial `ObservableList` to make updates.  If you don't have a reference to the original `ObservableList` you can get it from the `FilteredList` like this:

``` kotlin
Button("Add 3").apply {
    setOnAction {
        messages.value += "\n3 Button Clicked\n"
        (filterList.source as ObservableList<Int>).add(3)
    }
}
```
Note that this is backwards from instantiating an `ObservableList`, where you are instructed to stop updating the backing `List` directly, and only use the `ObservableList`.

### Changing the Predicate

You can change the `Predicate` on a `FilteredList` at any time.  Let's add a third `Button` to do this:

``` kotlin
Button("< 10").apply {
    setOnAction {
        messages.value += "\nPredicate Button Clicked\n"
        filterList.predicate = Predicate { item -> item < 10 }
    }
}
```
And it looks like this:
![Screen Snap 1]({{page.ScreenSnap1}})

Here the "Add 3" `Button` was clicked, followed by "Add 15" and then the `Predicate` change `Button`.  Then the other two `Buttons` were clicked again.  You can see that only values less than 10 are now shown in the `filterList ListView`.  

## SortedList

`SortedList` is very similar to `FilteredList` in the manner in which it works, but, of course, it sorts rather than filters.  Let's change our example to show this:

``` kotlin
class SortedListExample1 : Application() {

    private val obList: ObservableList<Int> = FXCollections.observableArrayList()
    private val sortedList = SortedList(obList, { a, b -> a.compareTo(b) })
    private val messages: StringProperty = SimpleStringProperty("")
    private val obTotalBinding = TotalBinding(obList)
    private val filterTotalBinding = TotalBinding(sortedList)

    override fun start(stage: Stage) {
        populateList(obList)
        obList.subscribe { messages.value += "obList Invalidated \n" }
        sortedList.subscribe { messages.value += "filterList Invalidated \n" }
        stage.scene = Scene(createContent()).addWidgetStyles()
        stage.show()
    }

    private fun createContent(): Region = BorderPane().apply {
        center = TextArea().apply {
            textProperty().bind(messages)
        }
        left = HBox(
            10.0,
            VBox(
                6.0,
                HBox(
                    20.0, promptOf("obList"),
                    Label().bindTo(obTotalBinding.asString())
                ) alignTo Pos.CENTER_LEFT,
                ListView<Int>(obList).apply { maxWidth = 100.0 }),
            VBox(
                6.0,
                HBox(
                    10.0, promptOf("filterList"),
                    Label().bindTo(filterTotalBinding.asString())
                ) alignTo Pos.CENTER_LEFT,
                ListView<Int>(sortedList).apply { maxWidth = 100.0 })
        )
        bottom = HBox(
            20.0,
            Button("Add 3").apply {
                setOnAction {
                    messages.value += "\n3 Button Clicked\n"
                    obList.add(3)
                }
            },
            Button("Add 15").apply {
                setOnAction {
                    messages.value += "\n15 Button Clicked\n"
                    obList.add(15)
                }
            },
        ) padWith 10.0
        padding = Insets(20.0)
    }

    private fun populateList(list: MutableList<Int>) {
        repeat(10) {
            list += Random.nextInt(20)
        }
    }
}

fun main() = Application.launch(SortedListExample1::class.java)
```
This is very similar to the first example.  We now have a `SortedList` with a `Comparator` specified instead of a `Predicate`.

A `Comparator` takes two values and returns either `1`, `0` or `-1` depending on whether or not the first value was greater than, equal to, or less than the second value.  In this case we are using `Int.compare()` as the `Comparator`.  

Also, in this example the values are randomly created between 1 and 20.  This is a little more interesting to look at.  

When running, it looks like this:

![Screen Snap 2]({{page.ScreenSnap2}})

You can see that "3,15,15,3" sequence at the bottom of the `obList ListView`, and you can see that `sortedList` integrates these new values in the correct order.

## Changing the Comparator

Just as with `FilteredList` and its `Predicate`, you can change the `Comparitor` at any time and the `SortedListed` will follow along.

To make it a little more interesting, I'll put in a crazy `Comparitor` - all of the odd numbers first (in order) and then all of the evens, also in order:

``` kotlin
Button("Odds and Evens").apply {
    setOnAction {
        messages.value += "\nComparator Button Clicked\n"
        sortedList.comparator = Comparator { a, b ->
            if ((a % 2) != 0) {
                if ((b % 2) != 0) a.compareTo(b)
                else -1
            } else {
                if ((b % 2) != 0) 1
                else a.compareTo(b)
            }
        }
    }
}
```
This is what it looks like before the "Odds and Evens" `Button` is clicked:

![Screen Snap 3]({{page.ScreenSnap3}})

and after...

![Screen Snap 4]({{page.ScreenSnap4}})

One of the things you should make note of here is that final "sortedList Invalidated" message, that occurred *after* the "Odds and Evens" `Button` was clicked.  That means that the `TransformationLists` invalidate when their predicate/comparitor changes.
# Predicate and Comparator as Properties

If you've looked at the JavaDocs for `SortedList` and `FilteredList` you might have noticed that the `Comparator` and the `Predicate` are implemented as `Properties`.  

This is interesting because it means that you can put `Listeners` on them, and detect when they have changed.  You can also put `Bindings` on them.  This is especially interesting for a number of reasons:

1. You can have your filtering/sorting logic split out from the implementation of your `ObservableLists`.
1. You can have several `ObservableLists` connected together in their filtering/sorting criteria.

Let's try adding a `Spinner` to control the threshold in the comparator of our `FilteredList`:

``` kotlin
class FilterListExample2 : Application() {

    private val obList: ObservableList<Int> = FXCollections.observableArrayList()
    private val filterList = FilteredList(obList) { item -> item > 6 }
    private val messages: StringProperty = SimpleStringProperty("")
    private val obTotalBinding = TotalBinding(obList)
    private val filterTotalBinding = TotalBinding(filterList)
    private val threshold: IntegerProperty = SimpleIntegerProperty(5)

    override fun start(stage: Stage) {
        populateList(obList)
        obList.subscribe { messages.value += "obList Invalidated \n" }
        filterList.subscribe { messages.value += "filterList Invalidated \n" }
        filterList.predicateProperty().bind(ThresholdBinding(threshold))
        stage.scene = Scene(createContent()).addWidgetStyles()
        stage.show()
    }

    private fun createContent(): Region = BorderPane().apply {
        center = TextArea().apply {
            textProperty().bind(messages)
        }
        left = HBox(
            10.0,
            VBox(
                6.0,
                HBox(
                    20.0, promptOf("obList"),
                    Label().bindTo(obTotalBinding.asString())
                ) alignTo Pos.CENTER_LEFT,
                ListView<Int>(obList).apply { maxWidth = 100.0 }),
            VBox(
                6.0,
                HBox(
                    10.0, promptOf("filterList"),
                    Label().bindTo(filterTotalBinding.asString())
                ) alignTo Pos.CENTER_LEFT,
                ListView<Int>(filterList).apply { maxWidth = 100.0 })
        )
        bottom = HBox(
            20.0,
            promptOf("Threshold"),
            Spinner<Int>(0, 20, 5).apply {
                threshold.bind(this.valueProperty())
            }
        ) padWith 10.0
        padding = Insets(20.0)
    }

    private fun populateList(list: MutableList<Int>) {
        for (x in 0..10) {
            list += x
        }
    }
}

class ThresholdBinding(private val threshold: ObservableIntegerValue) : ObjectBinding<Predicate<Int>>() {
    init {
        super.bind(threshold)
    }

    override fun computeValue(): Predicate<Int> = Predicate { item -> (item > threshold.get()) }
}

fun main() = Application.launch(FilterListExample2::class.java)
```
There's a new `IntegerProperty` called `threshold`.  There's also a new `ObjectBinding<Predicate<Int>>` called `ThresholdBinding`.  This `Binding` takes an `ObservableIntegerValue` as a dependency, and returns a `Predicate<Int>`.  There's a `Slider` and the `threshold Property` is bound to its `valueProperty`.

When running, it looks like this:

![Screen Gif]({{page.ScreenSnap11}})

## ObjectBinding\<Predicate\>

The `ThresholdBinding` might be a little difficult to follow.  So let's take a look at it, starting with the idea of `Predicate`.

A `Predicate` is a functional interface that works very much like a `Function` in that it defines an object with a single method that takes one value, and returns another based on that input value.  In the case of a `Predicate` the value that is returned is always a `Boolean`.  That single, non-default, method is called `test()`.  Any implementation of `Predicate` needs to implement `test()`.  Here's an example:

``` kotlin
class SamplePredicate() : Predicate<Int> {

    override fun test(t: Int): Boolean {
        return if ((t == 1) || (t == 7)) true else false
    }
}
    .
    .
    .
val samplePredicate = SamplePredicate()    
```
If we want to implement this logic as an anonymous inner class, we'd do it like this:

``` kotlin
val samplePredicate = object : Predicate<Int> {
    override fun test(t: Int): Boolean {
        return (t == 1) || (t == 7)
    }
}
```
And, of course, if we can do this, then we can implement it as a lambda function:

``` kotlin
val samplePredicate = Predicate<Int> { t -> (t == 1) || (t == 7) }
```
All three version are very similar, and the last two are essentially identical.  You can see, though, that all we need to do to define a `Predicate` is to provide the type of the value to be tested, and the `test()` method itself.  

What you need to understand is that `samplePredicate` in these examples is a variable that holds an object which is a subclass of `Predicate`.  `Predicate` itself, is just an object without any data fields, but a single method.  You can treat `samplePredicate` like data, it's just data that happens to only have a method.

When we have an `ObjectProperty<Predicate>` we are just treating that `Predicate` like data.  It's no different from `ObjectProperty<Customer>` or `ObjectProperty<String>` in this respect.  We can put values into it, and we can `Bind` it to other `Properties`, which is what we have in our latest example program:

``` kotlin
class ThresholdBinding(private val threshold: ObservableIntegerValue) : ObjectBinding<Predicate<Int>>() {
    init {
        super.bind(threshold)
    }

    override fun computeValue(): Predicate<Int> = Predicate { item -> (item > threshold.get()) }
}
```
This is a `Binding` that is dependent upon an `ObservableIntegerValue` called `threshold`, and that returns a `Predicate<Int>`.  This means that every time that the `threshold` changes, `computeValue()` will be called and return a new `Predicate` based on the current value of `threshold`.  For instance, if the value of `threshold` changes to "7", then the `Predicate` that is generated will be `{item -> item > 7}`, and if it changes to "25" then the `Predicate` will be regenerated to `{item -> item > 25}`.

Presumably, every time that the `FilteredList.predicateProperty()` changes, the `FilteredList` will be triggered to run every item in the backing `ObservableList` against the new `Predicate` which may change the contents of the `FilteredList`.

You need to be comfortable with the idea of a piece of code which is wrapped into an object, which in turn is generated by a piece of code which is wrapped inside another object which is dependent on some other data elements.

## Binding the Comparator

I thought I would do something a little different for this one...

``` kotlin
class SortedListExample2 : Application() {

    private val obList: ObservableList<Int> = FXCollections.observableArrayList()
    private val sortedList = obList.sorted({ a, b -> a.compareTo(b) })
    private val messages: StringProperty = SimpleStringProperty("")
    private val obTotalBinding = TotalBinding(obList)
    private val filterTotalBinding = TotalBinding(sortedList)


    override fun start(stage: Stage) {
        populateList(obList)
        obList.subscribe { messages.value += "obList Invalidated \n" }
        sortedList.subscribe { messages.value += "sortedList Invalidated \n" }
        sortedList.comparatorProperty().bind(ComparatorBinding(obList))
        stage.scene = Scene(createContent()).addWidgetStyles()
        stage.show()
    }

    private fun createContent(): Region = BorderPane().apply {
        center = TextArea().apply {
            textProperty().bind(messages)
        }
        left = HBox(
            10.0,
            VBox(
                6.0,
                HBox(
                    20.0, promptOf("obList"),
                    Label().bindTo(obTotalBinding.asString())
                ) alignTo Pos.CENTER_LEFT,
                ListView<Int>(obList).apply { maxWidth = 100.0 }),
            VBox(
                6.0,
                HBox(
                    10.0, promptOf("sortedList"),
                    Label().bindTo(filterTotalBinding.asString())
                ) alignTo Pos.CENTER_LEFT,
                ListView<Int>(sortedList).apply { maxWidth = 100.0 })
        )
        bottom = HBox(
            20.0,
            Button("Add 3").apply {
                setOnAction {
                    messages.value += "\n3 Button Clicked\n"
                    obList.add(3)
                }
            },
            Button("Add 15").apply {
                setOnAction {
                    messages.value += "\n15 Button Clicked\n"
                    obList.add(15)
                }
            }
        ) padWith 10.0
        padding = Insets(20.0)
    }

    private fun populateList(list: MutableList<Int>) {
        repeat(10) {
            list += Random.nextInt(20)
        }
    }
}

fun main() = Application.launch(SortedListExample2::class.java)
```
Here's the `ComparatorBinding` that it uses:

``` kotlin
class ComparatorBinding(private val list: ObservableList<Int>) : ObjectBinding<java.util.Comparator<Int>>() {

    init {
        super.bind(list)
    }

    override fun computeValue(): java.util.Comparator<Int> {
        return if (list.size > 10) Comparator { a, b ->
            if ((a % 2) != 0) {
                if ((b % 2) != 0) a.compareTo(b)
                else -1
            } else {
                if ((b % 2) != 0) 1
                else a.compareTo(b)
            }
        } else Comparator { a, b -> a.compareTo(b) }
    }
}
```
This takes the `ObservableList` itself as the dependency, and it uses the `List` size to determine which `Comparator` will be returned.  If the `ObservableList` has more than 10 items, then it will use the "Odds, Then Evens" `Comparator`, otherwise it just sorts them in numeric order.

I like this because it's very self-contained and self-referential.  The `SortedList` is designed to behave differently depending on some aspect of the list, in this case, its size.

# Sorted & Filtered Lists

You can stack `SortedList` into `FilteredList` and visa versa.  


# Composed List Items

Both `SortedList` and `FilteredList` get a little more interesting when the `List` items are objects composed of multiple data fields.  

Let's imagine an object like this:

``` kotlin

```

## List Items Composed of Properties

What if you change a value in one of the fields in a `FilteredList`/`SortedList` item?  Will the `List` change?
