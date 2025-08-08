---
title:  "ObservableLists: Extractors"
date:   2025-08-05 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/extractors
PantsOnFire: /assets/images/PantsOnFire.png
ScreenSnap0: /assets/elements/Extractors0.png
ScreenSnap1: /assets/elements/Extractors1.png
ScreenSnap2: /assets/elements/Extractors2.png
ScreenSnap3: /assets/elements/Extractors3.png
ScreenSnap4: /assets/elements/Extractors4.png
ScreenSnap5: /assets/elements/Extractors5.png
ScreenSnap6: /assets/elements/Extractors6.png
ScreenSnap7: /assets/elements/Extractors7.png
ScreenSnap8: /assets/elements/Extractors8.png
ScreenSnap9: /assets/elements/Extractors9.png
ScreenSnap10: /assets/elements/Extractors10.png
ScreenSnap11: /assets/elements/Extractors11.png
ScreenSnap12: /assets/elements/Extractors12.png
ScreenSnap13: /assets/elements/Extractors13.png

Diagram: /assets/elements/ListProperties.png
OLArticle: /javafx/elements/observable-classes-lists
OBGuide: /javafx/elements/observables_guide

JavaDocOL: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/ObservableList.html
JavaDocFXC: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/FXCollections.html

excerpt: ObservableList extractors allow you to track changes to elements inside your ObservableLists.
---

# Introduction

`ObservableLists` (and `ObservableSets`, and `ObservableMaps`, for that matter), are designed to invalidate and trigger `Listeners` when items are add or removed from them.  Screen elements like `ListView`, `TableView` and the pop-ups in `ComboBox` will automatically update when the `ObservableLists` that back them are updated.

But what about those cases where your `ObservableLists` are made of up composed objects?  Is it possible to detect that an `ObservableList` has changed when one of the fields inside an item has changed?

That's what "extractors" are for.  They allow you to detect changes to `Property` fields inside the items in your list, and to trigger `Listeners` on the `ObservableList` as a result.

# The Basics

The JavaDocs for extractors is pretty thin.

The first thing that you'll notice if you check out the entry for [ObservableList]({{page.JavaDocOL}}) is that there are no constructors for `ObservableList` because it's an `Interface`.  None of the "All Known Implementing Classes" section look promising either.

Typically, you'll get an `ObservableList` because it's already part of a `Node` class like `ListView`, `TableView` or `ComboBox`.  But if you want to create one yourself, you'll need to use the static library class, `FXCollections`.  

If you go to the JavaDocs for [FXCollections]({{page.JavaDocFXC}}) you'll see that there are lots and lots of ways to create different kinds of `ObservableLists`, some of which specify extractors.  Let's look at the details for `observableArrayList(Callback<E,Observable[]> extractor)`:

>**public static \<E\> ObservableList\<E\> observableArrayList(Callback<E,Observable[]> extractor)**
>
Creates a new empty ObservableList that is backed by an array list and listens to changes in observables of its items.
>
>The extractor returns observables (usually properties) of the objects in the created list. These observables are listened for changes and the user is notified of these through an update change of an attached ListChangeListener. These changes are unrelated to the changes made to the observable list itself using methods such as add and remove.
>
>For example, a list of Shapes can listen to changes in the shapes' fill property.

This seems to hint at something interesting, but it doesn't really explain much, does it?

Let's take a closer look at the constructor parameter: `Callback<E,Observable[]> extractor`.

A `Callback` is just a `Function`, meaning that it accepts one value and then returns another.  The name "Callback" implies that it is going to be used somewhere deep down in the internals of something.  Somewhere that we can't see, and probably don't want to see.  The `Callback` is a "hook" to allow that hidden code to get at something that we, the application programmers, define.  

In this case, the `Callback` accepts something of type `E`.  What is `E`? Well, `ObservableList` itself is generic, and `E` in this case refers to the type of elements that comprise our `ObservableList`.  This means that the `Callback` is just going to accept a single element of our `ObservableList`, whatever that happens to be.

The output from our `Callback` is going to be an `array` of `Observable`.  `Observable` is the top level `Interface` that all of the `Properties` and observable classes implement.  It only specifies three methods, `addListener()`, `removeListener()` and `subscribe()`.  That last one, `subscribe()` is new, and `ObservableList` extractors pre-dates it.  So we don't need to worry about it here.

What this tells us is that at some point, deep inside the `ObservableList` code, JavaFX is going to add a `Listener` to every `Observable` that we return for every item in our `ObservableList`.

The only question left is: What does that `Listener` do?

# What Does the Extractor Do?

In my article about [ObservableLists]({{page.OLArticle}}), part of my [series on Observables]({{page.OBGuide}}), I wrote some code to see what kind of `Listeners` are activated when changes are made to `ObservableLists`.  This involved adding, deleting, replacing and swapping `ObservableList` elements.  

We can use some of that same code here, to see how extractors translate to the `Listeners` triggered the `ObservableList`.

{% include notice_kotlin %}

To start, we'll look at the code without an extractor, and see what it does...

``` kotlin
private val obList: ObservableList<ExampleData> = FXCollections.observableArrayList()
private val messages: StringProperty = SimpleStringProperty("")
private val totBinding = object : IntegerBinding() {
    init {
        super.bind(obList)
    }

    override fun computeValue(): Int = obList.map { it.value2.value }.sum()
}

override fun start(stage: Stage) {
    obList.subscribe { messages.value += "Invalidated \n" }
    for (x in 1..5) obList.add(ExampleData(x))
    messages.value += "Data Loaded\n"
    messages.value += obList.map { it.value2.value }.joinToString() + "\n"
    stage.scene = Scene(createContent()).apply { }
    stage.show()
}

private fun createContent(): Region = BorderPane().apply {
    top = HBox(
        10.0,
        Label().apply { textProperty().bind(totBinding.asString()) },
        Label().apply { textProperty().bind(obList[2].value2.asString()) })
    center = TextArea().apply {
        textProperty().bind(messages)
    }
    bottom = Button("Increment Item").apply {
        setOnAction {
            messages.value += "Button Clicked\n"
            with(obList[2]) { value2.value = value2.value + 1 }
            messages.value += obList.map { it.value2.value }.joinToString() + "\n"
        }
    }
    padding = Insets(20.0)
}
}

class ExampleData(initialValue: Int) {
    val value1: IntegerProperty = SimpleIntegerProperty(initialValue)
    val value2: IntegerProperty = SimpleIntegerProperty(initialValue)
}


fun main() = Application.launch(ExtractorExample1::class.java)
```
We have an `ObservableList` of `DataExample`, which is just an object with two `IntegerProperties` in it.  We populate it with 5 items, where both `IntegerProperties` are populated with the same value and that value increments for each item that we add.

The GUI is a `BorderPane` with an `HBox` at the top with two `Labels`.  One shows the total of all of `DataExample.value2` in the `ObservableList` through a `Binding` and the other has the current value of `obList[2].value2`.

The centre is a `TextArea` which is bound to a `StringProperty` called `messages`.  Various bits of code add new data to `messages`, and we'll look at this.

At the bottom is a `Button`.  When this `Button` is clicked three things happen:

1. "Button Clicked" is added to `messages`.
1. `obList[2].value2` is incremented.
1. A list of all of the values in `value2` is added to `messages`

We also have an `InvalidationListener` (through `subscribe()`) on `obList` that just adds "Invalidated" to `messages`.  And we add "Data Loaded" to `messages` when all of the setup is done.

And all of this looks like this:

![Screen Capture 2]({{page.ScreenSnap2}})

You can see that `obList` invalidates every time a new `ExampleData` is added to it.  Then the "Data Loaded" message is appended and we see the initial list of values in `value2`.

Then we see that the `Button` was clicked, and that `obList[2].value2` was incremented.  And then again, and again, and again.

We can also see that our `Label` bound to `obList[2].value2` has been updated, and it shows "7" - just as you would expect.

However, the other `Label` hasn't been updated.  This is the one bound to the `ObservableList` itself.  Also, out `InvalidationListener` never fired again.

{% include notice type="primary" content = "When ObservableList items are composed objects containing Observable type fields, changes to those internal fields will not trigger Listeners on the ObservableList." %}

## Adding an Extractor

All we are going to do is change a single line of the code from above.  This:

``` kotlin
private val obList: ObservableList<ExampleData> = FXCollections.observableArrayList()
```
will change to this:
``` kotlin
private val obList: ObservableList<ExampleData> =
    FXCollections.observableArrayList({ item -> arrayOf(item.value2) })
```
In this code, `{ item -> arrayOf(item.value2) }` is the Callback for the extractor.  It creates an array, and it includes `ExampleData.value2` in that array.

Now, when we run the program and click the `Button` four times, we get this:

![Screen Capture 3]({{page.ScreenSnap3}})

The first thing that you notice is that we get the "Invalidated" message right after every "Button Clicked" message.  This means that the `InvalidationListener` on `obList` has fired!

The other thing that you should notice is that the first `Label` now says "19", which is 1+2+7+4+5.  This means that the `Binding` on `obList` that sums up the values is now working!

{% include notice type="primary" content = "When an ObservableList has an extractor that returns an encapsulated Observable field of the List items, then changes to that internal fields will trigger Listeners on the ObservableList." %}

This tells us that, at a minumum, the `InvalidationListener` that the `ObservableList` puts on every `value2` of every `ExampleData` in the list is used to trigger an `InvalidationListener` on the `ObservableList` itself.  That's enough to trigger the `Binding` on the `ObservableList`, too.

But does it do more?

# Will a ListChangeListener Work?

Let's change the code a little bit to add a `ListChangeListener` to `obList` to see if the extractor will cause it to fire:

``` kotlin
override fun start(stage: Stage) {
    obList.subscribe { messages.value += "Invalidated \n" }
    obList.addListener(ListChangeListener { change ->
        while (change.next()) {
            if (change.wasPermutated()) {
                messages.value += "List was permutated\n"
            } else {
                if (change.wasRemoved()) {
                    messages.value += "List item was removed\n"
                }

                if (change.wasAdded()) {
                    messages.value += "List item was added\n"
                }
            }
        }
    })
    for (x in 1..5) obList.add(ExampleData(x))
    messages.value += "Data Loaded\n"
    messages.value += obList.map { it.value2.value }.joinToString() + "\n"
    stage.scene = Scene(createContent()).apply { }
    stage.show()
}
```
`ListChangeListener` accepts a value of `ListChangeListener.Change`, which itself is an iterable construct and may contain several different changes to the `ObservableList` that happened at the same time.  These changes can be that an item (or range of items) were removed, added or permutated (which means they were replaced).  We need code that will look at all of the types of changes to see what is happening.

Again, we'll run the code and click on the `Button` four times:

![Screen Capture 4]({{page.ScreenSnap4}})

Here, we can see that the `ListChangeListener` fires as each item is added to the `ObservableList`, but then it doesn't fire again.

From this, we can safely assume that a `ListChangeListener` is never going to detect a change made to a field specified in an extractor.

# Will a ListProperty Work?

There is another class, called `ListProperty` that acts as a wrapper around an `ObservableList`.  What if we put our `obList` into a `ListProperty` and then add a `ChangeListener` to that `ListProperty`?

``` kotlin
override fun start(stage: Stage) {
    val listProperty = SimpleListProperty(obList)
    listProperty.addListener(InvalidationListener { messages.value += "Invalidated \n" })
    listProperty.subscribe { oldVal, newVal ->
        messages.value += "Change -> Old: $oldVal  New: $newVal\n"
    }
    for (x in 1..5) obList.add(ExampleData(x))
    messages.value += "Data Loaded\n"
    messages.value += obList.map { it.value2.value }.joinToString() + "\n"
    stage.scene = Scene(createContent()).apply { }
    stage.show()
}
```
Notice that I've also moved the `InvalidationListener` to the `ListProperty`, so that we can see if it fires on there as well.

When we run this, clicking the `Button` four times, we get:

![Screen Capture 5]({{page.ScreenSnap5}})

We see the `ChangeListener` firing as the elements are added, and this not suprising.  You will notice, however, thet the `oldVal` and `newVal` are both actually the `newVal`.  This is a quirk of `ChangeListeners` on `ListProperty`, and has nothing to do with extractors.  

Then we see the `ChangeListener` also fires on the changes to the enclosed field through the extractor.  This is interesting, although we still don't get to see the correct value for `oldVal`.  

You should also note that the `InvalidationListener` also fires every time for the `ListProperty`.

And just to make sure this is the extractor that is doing this, I took it out and re-ran the program:

![Screen Capture 6]({{page.ScreenSnap6}})

Here, we just get the `Listeners` firing for the additions to the `ObservableList`.  The changes to the enclosed `Property` do not trigger any `Listeners`.

# Advanced Extractors

If you look up extractor examples on-line, you'll never see anything more than:

``` java
Callback<TestClass, Observable[]> extractor = obj -> new Observable[]{ obj.nameProperty() };
ObservableList<TestClass> list = FXCollections.observableArrayList(extractor);
```
I just pulled this example from a StackOverflow question, but it's pretty much the standard.

But the extractor is a `Callback` and can do much more, if you want.  Let's take a look at that:

``` kotlin
class ExtractorExample4 : Application() {

    private val obList: ObservableList<ExampleData1> =
        FXCollections.observableArrayList({ item -> item.extractableValues() })

    private val messages: StringProperty = SimpleStringProperty("")
    private val totBinding = object : IntegerBinding() {
        init {
            super.bind(obList)
        }

        override fun computeValue(): Int = obList.map { it.value2.value }.sum()
    }

    override fun start(stage: Stage) {
        obList.addListener(InvalidationListener { messages.value += "Invalidated \n" })
        for (x in 0..5) obList.add(ExampleData1(x))
        messages.value += "Data Loaded\n"
        messages.value += obList.map { it.value2.value }.joinToString() + "\n"
        stage.scene = Scene(createContent()).apply { }
        stage.show()
    }

    private fun createContent(): Region = BorderPane().apply {
        top = HBox(
            10.0,
            Label().apply { textProperty().bind(totBinding.asString()) },
            Label().apply { textProperty().bind(obList[2].value2.asString()) })
        center = TextArea().apply {
            textProperty().bind(messages)
        }
        bottom = HBox(
            20.0, Button("Increment Item 2").apply {
                setOnAction {
                    messages.value += "Button 1 Clicked\n"
                    with(obList[2]) { value2.value = value2.value + 1 }
                    messages.value += obList.map { it.value2.value }.joinToString() + "\n"
                }
            },
            Button("Increment Item 5").apply {
                setOnAction {
                    messages.value += "Button 2 Clicked\n"
                    with(obList[5]) { value2.value = value2.value + 1 }
                    messages.value += obList.map { it.value2.value }.joinToString() + "\n"
                }
            })
        padding = Insets(20.0)
    }
}

class ExampleData1(private val initialValue: Int) {
    val value1: IntegerProperty = SimpleIntegerProperty(initialValue)
    val value2: IntegerProperty = SimpleIntegerProperty(initialValue)

    override fun toString() = "[${value1.value}, ${value2.value}]"

    fun extractableValues(): Array<Observable> {
        return if (initialValue < 4) arrayOf(value1) else arrayOf(value2)
    }
}


fun main() = Application.launch(ExtractorExample4::class.java)
```
You'll notice that we've added `ExampleData1`, which differs from `ExampleData` in two ways:

1. The constructor parameter `initialValue` has now been changed into a private immutable field.
1. We've have a new method, `extractableValues`.

This new method will return an `Array` containing either `value1` or `value2` depending on whether `initialValue` is less than 4 or not.

Additionally, we've added a second `Button` that works very much the same way as the original `Button` but acts on the 5th item in the `ObservableList`, which should have an `initialValue` greater than 4.

The `Listeners` have been return to very much the way that they were before we introduced the `ListProperty`, which has now been removed.

Here's what the screen looks like when we click the first `Button` a few times:

![Screen Capture 7]({{page.ScreenSnap7}})

You can see the messages from the `Button` click, and you can see the updated values in `obList`.  You can also see that the second `Label` is still correct.  However, the first `Label` does NOT have the correct value, and we do not have any "Invalidated" messages for those clicks.  

This is expected, because the `initialValue` of the item we clicked was "2" which is less than "4" and the extractor pulls `value1` for that item, while the `Button` is updating `value2`.

Here's what it looks like after we click the new `Button` a few times:

![Screen Capture 8]({{page.ScreenSnap8}})

Now we see the "Invalidated" messages, and the first `Label` now contains the correct total.

## Uses of This Technique

I've never seen anyone demonstrate anything like this before, and I'm not sure how often a use would pop up for this in real life.  But there are couple of things to take note of:

### The Extraction Details are Encapsulated

When you think about it, you see that this approach puts the logic for extraction *inside* the `ObservableList` items.  This means that the nature of the extraction is hidden from the `ObservableList` itself.  

If you are using a framework with a formal Presentation Model, like MVCI, then you are most likely going to instantiate your `ObservableLists` inside the Model.  Defining the extractors as part of that call to `FXCollections.observableArrayList()` requires that the Model knows about the structure of the objects that are put inside the `ObservableList`.

This might not be appropriate, even though those `ObservableList` items are part of the Presentation Model.  

Still, it's worth considering, and I can see some value in putting the logic that dictates how the items will behave in an `ObservableList` right beside the actual data that comprises the items.

### The Extraction Logic Needs to Be Immutable

You'll notice that I very deliberately based the extraction logic on a field defined as `val` instead of `var`.  This is the same as using `final` in Java.

This is because the extractor logic is only run once, when the item is added to the `ObservableList`.  

I checked the source code and found that the extractor code is run to get an `Array` of `Observable` and then the `ObservableList` loops through all of these `Observables` and adds an `InvalidationListener` to each one.  This code is run when the `ObservableList` is created as a wrapper around an existing `List`, like this:

``` kotlin
val obList = FXCollections.observableArrayList(existingList, extractor)
```
In this case, the interal code will loop through every item in `existingList`, run the `extractor` against them and the add the `InvalidationListeners` to every `Observable` returned by the `extractor` for each item.

The extractor code is also run when a new item is added to an existing `ObservableList` that has an extractor.  

There extractor code is also run when an item is removed from an `ObservableList`, in order to remove the `InvalidationListeners` that were added by the `ObservableList`.

Other than that, the extract isn't ever run against an item.  

If, in our example, we had made `initialValue` a var, and provided some way to change it after the items were in the list, there would be no change to `InvalidationListeners` added by the `ObservableList`, because they were already established.  So, you could end up with a system where the apparent behaviour of the extractor doesn't match the current state of the data.

The worse case scenario is that if the item is removed from the `ObservableList`, then some of the `InvalidationListeners` might not be removed because the extractor behaviour wasn't the same as when the item was added to the `ObservableList`.  That could give some weird results.

# Extractors in TableView Items

Probably the most common use of composed `Property` objects as items in `ObservableLists` is in `TableView`.  So let's take a look at how `TableView` columns are usually set up, and how they interact with enclosed `Property` fields.  

Here's the same example code, with just a `TableView` added:

``` kotlin
class ExtractorExample5 : Application() {

    private val obList: ObservableList<ExampleData> =
        FXCollections.observableArrayList()

    private val messages: StringProperty = SimpleStringProperty("")
    private val totBinding = object : IntegerBinding() {
        init {
            super.bind(obList)
        }

        override fun computeValue(): Int = obList.map { it.value2.value }.sum()
    }

    override fun start(stage: Stage) {
        obList.addListener(InvalidationListener { messages.value += "Invalidated \n" })
        for (x in 0..5) obList.add(ExampleData(x))
        messages.value += "Data Loaded\n"
        messages.value += obList.map { it.value2.value }.joinToString() + "\n"
        stage.scene = Scene(createContent()).apply { }
        stage.show()
    }

    private fun createContent(): Region = BorderPane().apply {
        top = HBox(
            10.0,
            Label().apply { textProperty().bind(totBinding.asString()) },
            Label().apply { textProperty().bind(obList[2].value2.asString()) })
        center = TextArea().apply {
            textProperty().bind(messages)
        }
        left = TableView<ExampleData>().apply {
            items = obList
            columns += TableColumn<ExampleData, Int>("Value 1").apply {
                setCellValueFactory { p -> p.value.value1.asObject() }
            }
            columns += TableColumn<ExampleData, Int>("Value 2").apply {
                maxHeight = 100.0
                setCellValueFactory { p -> p.value.value2.asObject() }
                setCellFactory { column ->
                    object : TableCell<ExampleData, Int>() {
                        override fun updateItem(newItem: Int?, empty: Boolean) {
                            super.updateItem(newItem, empty)
                            messages.value += "Table Cell Update: $newItem\n"
                            text = null
                            graphic = null
                            newItem?.let {
                                if (!empty) {
                                    text = newItem.toString()
                                }
                            }
                        }
                    }
                }
            }
        }
        bottom = Button("Increment Item 2").apply {
            setOnAction {
                messages.value += "Button 1 Clicked\n"
                with(obList[2]) { value2.value = value2.value + 1 }
                messages.value += obList.map { it.value2.value }.joinToString() + "\n"
            }
        }
        padding = Insets(20.0)
    }
}


fun main() = Application.launch(ExtractorExample5::class.java)
```
This is very close to the first example, but with the `TableView` added.  I've also removed the extractor from the `ObservableList`, so what we will see is pure `TableView` behaviour.

I added a custom `TableCell` to the second column so that we could see when the `updateItem()` method gets called, and what value it receives.  I've also limited the height of the `TableView` to restrict the number of `TableCells` instantiated because they generate a lot of messages, even if empty:

![Screen Snap 9]({{page.ScreenSnap9}})

The `Button` has been clicked twice, and you can see from the `TableView` that it has updated correctly.  You can see from the messages, that the `TableCells` get updated a lot just setting things up.  If you look at the bottom of the messages you can see the two `Button` clicks, and you can see the `TableCell` updating right away.  You do not see any invalidation messages from the `ObservableList`, however.

This tells us that the `TableView` puts a `Listener` of some sort on the `Property` when it identifies it by calling the `setCellValueFactory`.  That's the only way that this will work.  

What happens if we put the extractor back into the `ObservableList`?  You get this:

![Screen Snap 10]({{page.ScreenSnap10}})

Not much, really.

The first `Label` now has the correct total, as expected, and we see that the `ObservableList` invalidates after each `Button` click.  But it doesn't cause the `TableCell` to update twice, or anything like that.

We can determine from this that adding an extractor to an `ObservableList` used in a `TableView` doesn't hurt the performance of the `TableView`, but it doesn't help it either.

Of course this is only true **if** you compose your `TableView` items from `Properties` and other `ObservableValues`.  

# Extractors in ListView Items

So now we know that `TableView` works fine without extractors, but what about `ListView`?

We'll go back to our example, and change the layout so that it shows a `ListView` instead of a `TableView`:

``` kotlin
private fun createContent(): Region = BorderPane().apply {
    top = HBox(
        10.0,
        Label().apply { textProperty().bind(totBinding.asString()) },
        Label().apply { textProperty().bind(obList[2].value2.asString()) })
    center = TextArea().apply {
        textProperty().bind(messages)
    }
    left = ListView<ExampleData>().apply {
        items = obList
        cellFactory = Callback { _ ->
            object : ListCell<ExampleData>() {
                val label1 = Label()
                val label2 = Label()
                val layout = VBox(2.0).apply {
                    children += HBox(10.0, Label("Value 1:"), label1)
                    children += HBox(10.0, Label("Value 2:"), label2)
                    padding = Insets(8.0)
                }

                override fun updateItem(newItem: ExampleData?, isEmpty: Boolean) {
                    super.updateItem(newItem, isEmpty)
                    graphic = null
                    text = null
                    newItem?.let {
                        if (!isEmpty) {
                            label1.text = newItem.value1.value.toString()
                            label2.text = newItem.value2.value.toString()
                            graphic = layout
                        }
                    }
                }
            }
        }
    }
    bottom = Button("Increment Item 2").apply {
        setOnAction {
            messages.value += "Button 1 Clicked\n"
            with(obList[2]) { value1.value = value1.value + 1 }
            messages.value += obList.map { it.value1.value }.joinToString() + "\n"
        }
    }
    padding = Insets(20.0)
}
```
Here we have a custom `ListCell` layout that contains a `VBox` holding two `HBoxes`.  Each `HBox` has two `Labels`, one is just a title, and the other is loaded with either `ExampleData.value1.value` or `ExampleData.value2.value`.  No `Bindings` are used, so there's no reliance inside the `ListCell` on the `Property` nature of `value1` or `value2`.

The `updateItem()` method does the usual stuff, and loads `value1` and `value2` into their respective `Labels`.

The only other change to this code from the `TableView` version was to change the `Button` action to update `ExampleData.value1` instead of `ExampleData.value2`.  The `ObservableList` still has an extractor that returns `value2`, so this change essentially disables the extractor for this example.

This is what it looks like when it is run and the `Button` is clicked a few times:

![Screen Snap]({{page.ScreenSnap11}})

You can see that the values displayed in the `ListView` do not change at all.  You can see in the `TextArea` that the `Button` has been clicked several times, yet the `ObservableList` never invalidated, but that `value1` did increment each time.

If we change the extractor to return `value1` instead of `value2`, this is what it looks like:

![Screen Snap]({{page.ScreenSnap12}})

Now you can see that the `ListView` values are correct, and that the `Button` click causes the `ObservableList` to invalidate.

## Is This a Good Approach?

This approach has the advantage that you can implement a very naive design for the `ListCell`, without having to worry about binding and unbinding from a changing `itemProperty()`.  This is pretty much exactly the same approach to `ListCell` design that you'll see as the copypasta example in virtually every online tutorial about `ListView`.  It's simple, and it's easy, and it's what everybody knows.

However, it effectively splits the logic involved in keeping the `ListCell` up to date between two places, the `ListCell` itself, and the extractor.

{% include notice type="primary" content = 'Always ask yourself this question: "How much code will a maintenance programmer need to look at to fix a bug with this feature?"' %}

In this case, you cannot really understand how the `ListCell` works without looking at the instantiation of `ObservableList` to see that it has an extractor.  That's very likely to be in another class altogether.

Some time back, before JavaFX 19, achieving this entirely within the `ListCell` would have been a bit tedious.  The `Labels` would need to be bound to fields inside a value that itself was changing.  This would have required unbinding and the rebinding the `Labels` each time the value in the `ListCell` was changed.

But JavaFX 19 introduced `ObservableValue.flatmap()`, and now we can skip all the unbinding and rebinding.  

Here's the `ListCell` implemented that way:

``` kotlin
cellFactory = Callback { _ ->
    object : ListCell<ExampleData>() {
        val label1 = Label().apply {
            textProperty().bind(itemProperty().flatMap { item -> item.value1.asString() })
        }
        val label2 = Label().apply {
            textProperty().bind(itemProperty().flatMap { item -> item.value2.asString() })
        }
        val layout = VBox(2.0).apply {
            children += HBox(10.0, Label("Value 1:"), label1)
            children += HBox(10.0, Label("Value 2:"), label2)
            padding = Insets(8.0)
        }

        init {
            graphicProperty().bind(
                Bindings.createObjectBinding<Region>(
                    { if (item != null) layout else null },
                    itemProperty()
                )
            )
        }
    }
}
```
The `Labels` are bound to fields in `itemProperty()` through `flatMap()`.  Now, if the value in the field changes, or the `itemProperty()` itself changes, the binding will handle it automatically.  By also creating a `Binding` to control whether or not the `graphic` displays in the `init{}` block, we can now do away with `updateItem()` entirely.  Everything is simply dependent on `itemProperty()`.

In order to test this, I removed the extractor from the `ObservableList`.  The result looks like this:

![Screen Snap]({{page.ScreenSnap13}})

You can see that the items in the `ListView` are still correct, and the messages show the `Button` clicks that do **not** trigger an "Invalidated" message from the `Listener`.

Obviously, this requires a somewhat deeper understanding of the `ListCell` mechanics.  Most beginners aren't even aware that `item` exists in the `ListCell` as on `ObjectProperty`, and they assume that the only way to deal with a changing `item` is through `updateItem()`.  

# What About Performance?

You may be thinking, "What if I have an `ObservableList` with thousands, and thousands of items?  Won't that mean thousands and thousands of `Listeners` running all the time?"

For sure, there is some overhead to `Listeners`, but it's not as much as you might think.  Let's look at what happens when you add an `InvalidationListener` to an `Observable`...

To find this you need to look at `ObjectPropertyBase` which is the highest abstract class that implements `addListener()` for `Properties`.  In it, you'll find that it delegates to something called `ExpressionHelper` which is an instance variable of `ObjectPropertyBase`, and then that ends up in a subclass of `ExpressionHelper` called `Generic`.  In there, you will find this:

``` java
protected Generic<T> addListener(InvalidationListener listener) {
    if (invalidationListeners == null) {
        invalidationListeners = new InvalidationListener[] {listener};
        invalidationSize = 1;
    } else {
        final int oldCapacity = invalidationListeners.length;
        if (locked) {
            final int newCapacity = (invalidationSize < oldCapacity)? oldCapacity : (oldCapacity * 3)/2 + 1;
            invalidationListeners = Arrays.copyOf(invalidationListeners, newCapacity);
        } else if (invalidationSize == oldCapacity) {
            invalidationSize = trim(invalidationSize, invalidationListeners);
            if (invalidationSize == oldCapacity) {
                final int newCapacity = (oldCapacity * 3)/2 + 1;
                invalidationListeners = Arrays.copyOf(invalidationListeners, newCapacity);
            }
        }
        invalidationListeners[invalidationSize++] = listener;
    }
    return this;
}
```
Yikes!

In summary, it has an array of `InvalidationListeners` and it adds your new `InvalidationListener` to it.  The remaining dozen or so lines of appear to be some kind of memory management optimization.  That's good to see, if you are worried about performance degredation associated with extractors.

So, this means that every `Observable` potentially contains an `Array` that holds all of the `InvalidationListeners` that have been added to it.

How does it use this?

In `ObjectPropertyBase` we have this:

``` java
public void set(T newValue) {
    if (isBound()) {
        throw new java.lang.RuntimeException((getBean() != null && getName() != null ?
                getBean().getClass().getSimpleName() + "." + getName() + " : ": "") + "A bound value cannot be set.");
    }
    if (value != newValue) {
        value = newValue;
        markInvalid();
    }
}
```
The most important part for us is the call to `markInvalid()`:

``` java
protected void fireValueChangedEvent() {
    ExpressionHelper.fireValueChangedEvent(helper);
}

private void markInvalid() {
    if (valid) {
        valid = false;
        invalidated();
        fireValueChangedEvent();
    }
}
```
And we are back to `ExpressionHelper` again, and then back into `Generic` for the actual implementation:

``` java
for (int i = 0; i < curInvalidationSize; i++) {
    try {
        curInvalidationList[i].invalidated(observable);
    } catch (Exception e) {
        Thread.currentThread().getUncaughtExceptionHandler().uncaughtException(Thread.currentThread(), e);
    }
}
```
This is just a snippet of the code for this method, but you can see how it works.  It loops through the `Array` of `InvalidationListeners` and invokes `invalidated()` on each one.  Remember that `InvalidationListener` is a Functional Interface with one method, `invalidated()`.

If you were picturing an implementation where JavaFX had some master list of `Listeners` and it had to check them all every time something changed, then you can see that this is backwards to that.  Each `Observable` maintains its own list of `Listeners` that have been registered with it.  Then when its value is updated, it triggers each of those `InvalidationListeners`.  

If there is a potential performance hit, it would be from having many `Listeners` on each `Observable` item.  Not from having many `Observable` items with one or two `Listeners` each.

# Conclusion

Creating an `ObservableList` with an extractor causes just two things to happen:

1. The `ObservableList` will invalidate, and fire any `InvalidationListeners` on it when any `Observable` returned from the extractor invalidates.
1. If you wrap the `ObservableList` in a `ListProperty` that `ListProperty` will both invalidate and fire and `ChangeListeners` on it when any `Observable` return from the extractor invalidates.

Invalidation is **the** key concept on `Bindings`.  This is because `Bindings` always recalculate by calling their `computeValue()` method when any of their dependencies invalidates.  

This means that you can create `Bindings` on `ObservableLists` that will re-evaluate not just when items in the `ObservableList` are added, removed or replaced, but when internal values in those `ObservableList` items change...if you add the appropriate extractor.
