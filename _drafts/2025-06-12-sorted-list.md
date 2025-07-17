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

Diagram: /assets/elements/ListProperties.png
OLArticle: /javafx/elements/observable-classes-lists
OBGuide: /javafx/elements/observables_guide

JavaDocOL: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/ObservableList.html
JavaDocFXC: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/FXCollections.html

excerpt: "Explaining how to create and use two subclasses of ObservableList: FilteredListed and SortedList"
---

# Introduction

Both `FilteredList` and `SortedList` are "wrapper" classes that enclose an instance of `ObservableList`.  `FilteredList` allows you to add a `Predicate` to an `ObservableList`, which will only certain items to be seen.  `SortedList` allows you to add a `Comparator` to an `ObservableList`, which will determine the order in which the items will be seen.

The thing that makes these two classes different from just sorting a `List` or extracting the elements that you want into a second `List` is that they are "live".  The original `List`, being an `ObservableList` is continues to be watched, and changes in that `ObservableList` will be reflected in the `FilteredList`.

# The Basics

We are going to look at `FilteredList` and `SortedList` together because they share a great deal of their underlying concepts.

## TransformationList

The first of these is that they are both subclasses of the abstract class `TransformationList`.  Let's take a quick look at `TransformationList` so that we can understand these two implementations of it.

`TransformationList` has a public method called `getSource()` which allows you to get a reference to the enclosed `ObservableList`.  It also has two abstract public methods called `getViewIndex()` and `getSourceIndex()` which allow you to map item references between the `TransformationList` and its enclosed `ObservableList`.

Finally, we have the most interesting method, it's a protected abstract method called `sourceChanged(ListChangeListener.Change<? extends F> c)`.  Let's ignore the generic type stuff for now, and look at what this means.  It accepts a `ListChangeListener.Change` parameter.  So we now know that the `TransformationList` is kept synchronized with the underlying `ObservableList` by processing the individual changes to that `ObservableList`.  This might be useful in understanding how to manage `SortedList` and `FilteredList`.

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

Notice that all of the additions were done to `obList`, and yet both `Lists` were updated, if the `Predicate` was satisfied.

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
Note that this is backwards from instantiating an `ObservableList`, where you are to stop updating the backing `List` directly, and only use the `ObservableList`.

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

# Predicate and Comparator as Properties

If you've looked at the JavaDocs for `SortedList` and `FilteredList` you might have noticed that the `Comparator` and the `Predicate` are implemented as `Properties`.  

This is interesting because it means that you can put `Listeners` on them, and detect when they have changed.  You can also put `Bindings` on them.  This is especially interesting for a number of reasons:

1. You can have your filtering/sorting logic split out from the implementation of your `ObservableLists`.
1. You can have several `ObservableLists` connected together in their filtering/sorting criteria.
