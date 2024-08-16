---
title:  "Guide To the Observable Classes - Part III"
date:   2024-08-03 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/observable-classes-lists
ScreenSnap1: /assets/elements/ListObservables1.png
ScreenSnap2: /assets/elements/ListObservables2.png
ScreenSnap3: /assets/elements/ListObservables3.png
ScreenSnap4: /assets/elements/ListObservables4.png
Diagram: /assets/elements/ListProperties.png
part1: /javafx/elements/observable-classes-generics
part2: /javafx/elements/observable-classes-typed

excerpt: A look at the Observable classes that wrap ObservableList
---

# Introduction

In [Part 1]({{page.part1}}) of this series, we looked at the `Observable` classes designed to wrap aribitrary `Object` values, and in [Part II]({{page.part2}}) we looked at the typed `Observable` classes.  In this article we are going to look at the `Observables` designed to hold `ObservableLists`.

This is a subject that always confused me.  Aren't `ObserableLists` already `Observable`?  And doesn't `ObseravbleList` already implement the `Obseravble` interface?  So why wrap an `ObservableList` inside an `Observable` wrapper?

The whole thing, at first glance, seems very circular.  We have an `Observable` that wraps an `ObservableList` and supports these same methods as `ObservableList`.  What's the point?

The truth is, you've probably already used a `Property` that wraps an `ObseravbleList` if you have ever used `TableView`.  That's because the `items` field in `TableView` is an `ObjectProperty<ObservableList<S>>`.  But it's highly unlikely that you've ever used this as a `Property`, meaning that you've probably never put a `Listener` on it or bound it to or from anything.

We should have a closer look at what this does.

# ObservableList Observables in Action

The best way to figure the value in `List Obseravbles` is to see one in action...

## The Basic Scenario

Here's a small program that can test how `ObjectProperty<ObservableList<String>>` works:

``` kotlin
class Example1 : Application() {
    @Throws(Exception::class)
    override fun start(primaryStage: Stage) {

        val mainScene = Scene(createContent(), 840.0, 700.0)
        primaryStage.scene = mainScene
        primaryStage.show()
    }

    private fun createContent(): Region = VBox(20.0).apply {
        val list1 = FXCollections.observableArrayList("abc", "def", "ghi")
        val list2 = FXCollections.observableArrayList("jkl", "mno")
        val messages = SimpleStringProperty("")
        val listProperty = SimpleListProperty(list1)
        listProperty.subscribe { oldVal, newVal ->
            messages.value += "Change -> Old: $oldVal  New: $newVal\n"
        }
        list1.subscribe { messages.value += "List1 Invalidated\n" }
        children += Label().apply {
            textProperty().bind(listProperty.sizeProperty().map { "List Size: $it" })
        }
        children += TextArea().apply {
            textProperty().bind(messages)
            prefHeight = 300.0
        }
        children += Button("Add Element to List1").apply {
            setOnAction {
                messages.value += "Add Element to List1 clicked\n"
                list1.add("xyz")
            }
        }
        children += Button("Swap to List2").apply {
            setOnAction {
                messages.value += "Swap List clicked\n"
                listProperty.set(list2)
            }
        }
        padding = Insets(30.0)
    }
}


fun main() = Application.launch(Example1::class.java)
```
Data-wise, we have two `ObservableArrayLists<String>` called `list1` and `list2`, and a `SimpleListProperty` called `listProperty`.  We start off with `list1` loaded into `listProperty`.  

On the screen, we have a `Label()` with its `textProperty` bound to  `listProperty.sizeProperty()`.  Then we have a `TextArea` with its `textProperty` bound to a `StringProperty` called `messages`.

We've added a `Subscription` to the `listProperty` to add a line to `messages` whenever its `value` changes.  Then we've added a `Subscription` to `list1` to tell us whenever it becomes invalidated.  

Finally, we have two `Buttons`.  The first adds a line to `messages` and adds a new element to `list1`.  The second swaps `list2` into `listProperty` after adding a line to `messages` so that we can tell when it was clicked.  

Let's see what happens.  First we'll click the first `Button` a couple of times:

![Screen Snap 1]({{page.ScreenSnap1}})

You can see that the list size of 5 matches the size of `list1` which is still loaded into `listProperty`.  You can see the two messages from clicking the first `Button` twice, and you can see the two "Invalidation" messages from the `Listener` on `list1`.  

The really interesting part is that you can see the messages from the `Subscription` on `listProperty`.  But we didn't change the `value` of `listProperty`!

This tells us the changes to the `ObseravbleList` in a `ListProperty` trigger a change in the `ListProperty` itself!  But you can see that both `oldVal` and `newVal` are still the same thing.  So it's not like we get a snapshot of what the `ObservableList` looked like before and after the change.  

Then we'll click on the second `Button` and then the first `Button` a few more times:

![Screen Snap 2]({{page.ScreenSnap2}})

You can see the message where the second `Button` was clicked, and then the `Subscription` message.  Now we have different values for `oldVal` and `newVal`.  You can also see that the `listProperty.sizeProperty()` has changed from the `Label` at the top.

Next the first `Button` was clicked twice more, and you can see the messages from the `Invalidation Subscription` on `list1`.  Interestingly, even though we haven't done any `getValue()` operations on `list1` it seems to be revalidating itself, because the second click also triggered the `Subscription`.

## Adding a ListChangeListener

There's another thing we can try.  `ListProperty` supports `ObseravbleList.addListener(ListChangeListener)` which will show us actual changes.  

Let's see if we swap out the `Subscription` on `listProperty` with one of those:

``` kotlin
listProperty.addListener(ListChangeListener { change ->
    messages.value += "listProperty change: $change \n"
})
```
And we'll add another `Button` to add an element to `list2`:

``` kotlin
children += Button("Add Element to List2").apply {
    setOnAction {
        messages.value += "Add Element to List2 clicked\n"
        list2.add("xyz")
    }
}
```
And this is what happens:

![Screen Snap 3]({{page.ScreenSnap3}})

We start off by clicking on the first `Button` and we see `list1` invalidated and the `ListChangeListener` on `listProperty` fires and adds its message.  Then we click on the `Button` to add an element to `list2`, which generates no new messages other than the `Button` click message.  We click the first `Button` again, and see all the messages that we expect to see.

Then we click the "Swap" `Button` and we see the `ListChangeListener` on `listProperty` fire again!  It tells us that the entire `ObseravbleList` has been changed by showing us the elements (all of them) being replaced.  Notice that this is fundamentally different from the way that the old `Subscription` worked by showing us the change in the wrapper `value`.

Next, the first `Button` was clicked again.  This generated the click message, and the `list1` invalidation message, but no messages from the `ListChangeListener`.  So we can see that the `ListChangeListener` didn't just get passed on down to `list1` through `listProperty`.

Finally, the `Button` to add an element to `list2` was clicked.  We see the click message, and then a message from the `ListChangeListener` on `listProperty`, telling us that a new element was added to `listProperty`.

## Adding Elements to the Wrapper Observable

`ListProperty` also supports the `List.add()` method.  What happens if we try that.  

We'll change the "Add to List1" `Button` to add to `listProperty` instead.  Then we'll take away "Add to List2" and put an `Invalidation Subscription` on `list2`:

``` kotlin
children += Button("Add Element to listProperty").apply {
    setOnAction {
        messages.value += "Add Element to listProperty clicked\n"
        listProperty.add("xyz")
    }
}
```
Let's see what happens:

![Screen Snap 4]({{page.ScreenSnap4}})

You can see the "Add" `Button` is clicked twice, and we get the messages that we would expect; `list1` is invalidated and the change on `listProperty` is shown both times.  

Then the "Swap" `Button` is clicked and we see the click message and the change of all of the elements in `listProperty` displays.  No `InvalidationListeners` fire.

Finally the "Add" `Button` is clicked twice more and we see that `list2` is invalidated and the change on `listProperty` displays twice.

## What This Means

In our examples, we see how a `ListProperty` acts exactly like a `Property<ObseravbleList>` but also exactly like the `ObservableList` that is currently in its `value`.

This explains exactly why `TableView` has its `items` field as a `ListProperty`.  Internally, it can establish all of it's relationships and `Listeners` to the `items` through the `ListProperty`.  Then, when some external code swaps out `items` for some other `ObservableList` all off those relationships and `Listeners` will automatically switch over to the new `ObservableList` without any need to code anything.  

If it wasn't like this, if you wanted to swap out the list of items in a `TableView`, you would have clear out `items` and then load the elements of your new `List` into it.

Now let's take a look at how all of the classes and elements hook up together...

# The List Classes and Interfaces

We'll start off with the chart, which looks quite a bit like the charts from the first two parts of this series:

![List Properties]({{page.Diagram}})

Here you can see that all of the generic `<T>`'s have been replaced with `<ObservableList<E>>` and the typed entries are of `<E>`.  I've used `ObL` as a shorthand for `ObservableList<E>` to avoid having to use a tiny, tiny font.

Understand, though, that there is no interface called `ObservableValue<ObservableList<E>>`, it's just `ObservableValue<T>` with the `<T>` resolved as a `<ObservableList<E>>`.  The same holds true for all of the other entries on the chart defined with `<ObL>`.

This chart is a little bit more complicated in its relationships because of the dual base interfaces; `ObservableValue` and `ObservableList`.  However, you can see that every entry with "List" in its name implements `ObservableList`, while those without it do not.  For instance, `WritebleListValue` implements `ObservableList`, while `Property` does not.  

## The Interfaces

We'll start with the interfaces.  Some of the descriptions are a little bit stripped down from what you'll see in [Part 1]({{page.part1}}).  I want this article to be complete within itself, but I don't want to repeat a lot of information that you can find in [Part 1]({{page.part1}}).

### Observable & ObservableValue\<ObservableList\<E\>\>

At the top of the chart, you'll find the interface, `Observable`.  You'll notice that it is not typed at all, it's not even a generic.  That's because all of its methods do not deal with data.

This is the root for all of the other classes and interfaces, but it's really quite simple and only defines three methods: `addListener()`, `removeListener()` and `subscribe()`.  These are all related to the process called "Invalidation".

The next interface down is `ObservableValue<ObservableList<E>>`, which extends `Observable`.  The key method in this interface is `ObservableValue.getValue()`, which allows us to actually read the value and will re-validate the `Observable`.

Now that we can read the value we can have the methods related to `ChangeListeners`, including those related to `Subscriptions`.  The other methods it adds are the `map()` and `flatMap()` methods used for transforming an `ObservableValue` into a different `ObservableValue`.

There is also the `ObservableObjectValue<ObservableList<E>>` interface, which introduces the `get()` method, which at this point is effectively identical to `getValue()`.

### ObservableListValue\<E\>

`ObservableListValue` is where the `List Observables` start to get very different from the other typed `Observables`.  This interface extends `ObservableObjectValue<ObservableList<E>>`, as expected, but it also implements `ObservableList<E>`.  Unlike, for instance, `ObservableStringValue`, it's not just a placeholder entry that you can treat the same as `ObservableObjectValue<ObservableList<E>>`.

This is the place on the chart where `ObservableValue` takes on the characteristics of  `ObservableList`.  In addition to the `getValue` and the two types of `Listener` methods we also `ListChangeListener` methods along with all of the methods in `List`, `Collection`, and `Iterable`.  There are some extra methods for manipulating lists as well; `setAll()`, `addAll()`, and `removeAll()`.  

Finally, we get `filtered()` and `sorted()` which generate `FilteredList` and `SortedList`.

### ReadOnlyProperty\<ObservableList<E>\> & WritableValue\<ObservableList<E>\>

These are the two parent interfaces for all of the `Property` classes.  

`ReadOnlyProperty<ObservableList<E>>` simply introduces the `getBean()` and `getName()` methods.  These are presumably meaningful methods if you are serializing your `Properties`.

The interface `WritableValue<ObservableList<E>>` gives us the `setValue()` method.  Now we can actually put something into an `Observable`!

### WritableObjectValue\<ObservableList<E>\> and WritableStringValue

`WritabaleStringValue` is an extension of `WritableObjectValue<ObservableList<E>>` that adds no methods.  So you can treat it as a synonym for `WritableObjectValue<ObservableList<E>>`.

Note that these classes extend from `ObservableValue<ObservableList<E>>` and not `ObservableListValue`, so we don't have the `ObservableValue`/`ObservableList` duality here.

These two classes add the `set()` method, which, for `String` types is identical to `setValue()`.

### Property\<ObservableList<E>\>

The `Property` interface extends the previous two interfaces and adds the ability to bind to other observables.  We get `bind()` and `bindBidirectional()`, and their corresponding unbinding methods.  Also there is the `isBound()` method that will tell us if the `Property` has been bound to something.

### Binding\<String\>

`Binding<ObservableList<E>>` is the last interface on the chart.  It extends`ObservableValue<ObservableList<E>>` adding a method to force the `Binding` to invalidate, a method to check if the `Binding` is valid and a method to get the dependencies of the `Binding`.

## The Classes
Everything else on the chart is a class, let's take a look at them.

### StringExpression

This is the root class for all of the `Property<String>` classes, even though the diagram goes diagonally down to `ObjectProperty<String>`.  Note that `StringExpression` is a drop-in replacement for `ObjectExpression` and does not extend or implement `ObjectExpression`

This is the only class in the chart that introduces new concrete methods that don't implement methods already defined by any of the interfaces in the chart.

{% include notice type="primary" content = "StringExpression is where this chart really diverges from the generic chart.  Every class that inherits from StringExpression is fundamentally different from the generic equivalent." %}

This class provides the core functionality for the "Fluent API" for creating `String` bindings.  `StringExpression` is an abstract class that implements a number of `String` related operations such as `concat()` and `length()` and `isEmpty()` in an `Observable` fashion.  We also have methods for equality, inequality, greater than, less than, greater than or equal, less than or equal, and checking for a `Null` value, each of which creates a `BooleanBinding`.

There's also a method called `getValueSafe()`.  It returns a `String` and will return an empty `String` if the value in the `StringExpression` is `null`.

It is StringExpression that lets you do things like this:

``` kotlin
  var label = Label()
  label.textProperty().bind(model.lastNameProperty().concat(", ").concat(model.firstNameProperty()))
```

This inclusion of `StringExpression` in the class hierarchy is what makes `ObjectProperty<String>` different from `StringProperty`.  The code snippet above assumes that `model.lastNameProperty()` is a `StringProperty`, not an `ObjectProperty<String>`.  Because the latter wouldn't work because `ObjectExpression` (which is the base class for `ObjectProperty` does not define `concat()`.


### ReadOnlyStringProperty

This class extends `StringExpression` and implements `ReadOnlyProperty`.  This class can also be thought of as very much like `ReadOnlyProperty` but with the ability to use the Fluent API with it to create bindings.  

### StringProperty

The best way to think about `StringProperty` is an implementation of `Property<String>` but, since it inherits from `StringExpression` through `ReadOnlyStringProperty`, also has the methods for creating bindings via the Fluent API.  However, it only has implementations for bidirectional binding and setting the value.  

This is, however, a great class to declare your instantiated `Properties` as.  Like this:

``` kotlin
val someProperty : StringProperty= SimpleStringProperty("abc")
```

### StringPropertyBase

This class adds *almost* all the remaining methods defined in the various interfaces up hierarchy.  We get methods to bind and unbind, add and remove `Listeners` and a `get()` method.  

The methods that are missing are the two silly Java Bean related methods.  This is the class that you probably want to extend from if you want to create your own `Property` classes, especially if you want to ignore Java Bean stuff as much you can.

### SimpleStringProperty

This is the only class in the chart that you can directly instantiate, so everybody is familiar with it.  It adds the `getBean()` and `getName()` methods and the infrastructure and constructors to set their values.  

### ReadOnlyStringPropertyBase and ReadOnlyStringWrapper

`ReadOnlyStringPropertyBase` is very similar to `StringPropertyBase` in that it is an abstract class that has almost all of the interface methods implemented except for the "Bean" methods.  It's also missing the `get()` method.

The only "only out of the box" implementation of `ReadOnlyStringPropertyBase` is contained in `ReadOnlyStringWrapper`.  This is a fairly involved topic that was covered in depth [here]({{page.wrapper}}) in Part I.

### StringBinding

This is the sole class on the `Binding` side of the chart, and it's abstract and it extends `StringExpression`.  Any utility (including the Fluent API) that creates `String Bindings` will do so by extending from this class.

It doesn't add any new public methods but there are a few protected methods that are the framework for custom `Binding` classes.  We get `bind()` and `unbind()` to start with. We also get `allowValidation()`, `onInvalidating()` and `isObserved()`.  There is one abstract method, and that's `computeValue()` which is also protected.  

The intention is clear about the standard use case for custom classes extended from `StringBinding`.  It's that the `Observables` to be bound are defined when the class is instantiated and that you'll also define a `computeValue()` method that will use those bound values to determine the return value of any calls to `get()` or `getValue()`.

{% include notice type="warning" content = "Note that computeValue() is only called when a call is made to get() or getValue() when the Binding has been invalidated." %}

Since `StringBinding` extends from `StringExpression` you can use the Fluent API to modify the results or combine it with other `StringExpressions`.

## Compatibility Issues
The main difference between the entries on this chart and the one for the generic `Observables` is that `ObjectExpression` has been swapped out for `StringExpression`.  These two classes have different methods, which will create some type incompatibilities.

However, it turns out that you can fairly freely mix and match `StringProperty` and `ObjectProperty<String>` in bindings without a lot of problems.  

This is mostly because the `bind()` method in `ObjectPropertyBase<String>` takes an `ObservableValue<? extends String>`, while `StringPropertyBase.bind()` also takes `ObservableValue<? extends String>`.  And of course, both classes implement `ObservableValue<String>`.

You can even put a `ObjectProperty<String>` as a parameter in the Fluent API methods and it will work without a hitch.  Generally speaking, the Fluent API methods will accept primitive classes as well as `Observable` classes, but obviously won't trigger an invalidation if a primitive value changes.  Internally, a dependency is created if the parameter is an `instance of ObservableValue`.  Which of course, `ObjectProperty<String>` is.  

However, `ObjectProperty<String>` and `StringProperty` are **not** type compatible as method parameter declarations.  Unless you are planning on using the Fluent API do not create your method parameters as `StringProperty`.  The same goes for `ObjectProperty<String>` although you're probably not going to be using the methods from `ObjectExpression` over `StringExpression`.  Specify your parameters as `Property<String>` and it will accept both `ObjectProperty<String>` and `StringProperty`.

# Using this Information

At this point you should have a handle on when and how to use both the generic and the type `Observable` classes.  What are the some of the factors you need to keep in mind when using both the generic and the typed `Obsevables`?

## Converting Between Generic and Typed Properties

As we've seen `IntegerProperty` implements `Property<Number>`, not `Property<Integer>`.  This means that it also implements `ObservableValue<Number>`.  

And that means that you cannot pass `IntegerProperty` as a parameter when `Property<Integer>` or `ObservableValue<Integer>` is required.  

This quickly becomes frustrating.

Fortunately all of the `Property<Number>` implementations have a function called `asObject()`.  For `IntegerProperty` this will pass you `Property<Integer>` that is bidirectionally bound to your `IntegerProperty`.  It's not the same object cast differently, but an actual new object that's bidirectionally bound to the original.

All of these typed classes also have a static method that does the reverse.  For instance, `DoubleProperty` has a method called `doubleProperty()`.  You pass it a `Property<Double>` and it gives you back a new `DoubleProperty` that is bidirectionally bound to the original `Property<Double>`.  This goes for `StringExpression` and `BooleanExpression`

Knowing about this makes any issues about whether or not to use typed or generic `Properties` a lot less critical.  You can very easily convert from one to the other when required.

## Fluent API

The Fluent API is delivered via the `{Type}Expression` classes, and is the main motivation to use typed `Observable` classes over the generic ones.

Since the introduction of `ObservableValue.map()` in JFX 19, a lot of the utility in the Fluent API has become redundant, since because `map()` is simpler to use because the `Function` passed to it is just Java (or Kotlin) code.  What `ObservableValue.map()` cannot do, however, is to combine multiple `Observables` together as dependencies.

I find the Fluent API to be fine when the transformations are fairly simple.  But when the operations start to get long and involved, then the Fluent API tends to less clear.  And if it's less clear, then it's more likely to be the source of a bug or cause headaches in maintenance.  In those cases, it can be better to use a static methods from the `Bindings` class, or to create a `Binding` by extending one of the `{Type}Binding` classes.

In my opinion, then, the use case for the Fluent API is fairly narrow: Simple transformations involving two or more `Observables`.  

Not that this is uncommon.  Use cases like this happen all the time:

``` kotlin
val fullName : ObservableValue<String> = lastNameProperty.concat(", ").concat(firstNameProperty)
```
It's hard to think of an easier, cleaner way to do this.  However...
``` kotlin
val nameString : ObservableValue<String> = lastNameProperty.map{"The last name is $it"}
```
Cannot be performed via the Fluent API alone and you would have to otherwise use the `Bindings` library:
``` kotlin
val nameString : ObservableValue<String> = Bindings.concat("The last name is ").concat(lastNameProperty)
```
Note that the first `concat()` is from `Bindings` and the second is from `StringExpression`.

You can combine `ObservableValue.map()` with the Fluent API:
``` kotlin
val nameString : ObservableValue<String> = lastNameProperty.concat(", ").concat(firstNameProperty).map{"The full name is $it"}
```
But the `map()` call has to come at the end, because it returs `ObservableValue<String>` which doesn't support the Fluent API.  If you really did want to do it the other way:
``` kotlin
val nameString : ObservableValue<String> = StringExpression.stringExpression(lastNameProperty.map{"The full name is $it,"}).concat(firstNameProperty)
```
But, honestly, that seems like a lot of work.

## Method Overloading

In Java, you cannot overload methods where the signatures only differ by the types of their enclosed generics.  For instance this code will fail to compile:

``` kotlin
fun doSomething(param1 : Property<String>) {
   .
   .
}

fun doSomething(param1: Property<Int>) {
   .
   .
}
```
You get an error that says something about "erasure" and non-unique signatures.  What you can do instead is this:

``` kotlin
fun doSomething(param1 : StringProperty) {
   .
   .
}

fun doSomething(param1: IntegerProperty) {
   .
   .
}
```
This will work just fine.  This is the second reason that you might want to use the typed `Observable` classes.

## Dealing With Node Properties

You should be aware that virtually all of the JavaFX `Nodes` expose their `Properties` as typed `Properties` whenever possible.  For instance, `Region.widthProperty()` will return a `ReadOnlyDoubleProperty` while, `Region.paddingProperty()` will return `ObjectProperty<Insets>`.

This makes sense, since, as a general purpose API, they would want to provide the most "out of the box" functionality, and returning `Properties` that can be used in the Fluent API without any conversion seems like the way to go.  You'll have to decide for yourself if this would influence the way that you declare your own `Properties`.

There are a few places in the JavaFX API where it's hard to avoid understanding some of the nuances about the typed classes versus the generic classes.  For instance, `TextFormatter<Integer>` has a method `valueProperty()` that return `ObjectProperty<Integer>`, which is going to give you issues if you try to bidirectionally bind it with an `IntegerProperty` in your Model.  

## When to Use ReadOnly{Type}Property

If you look at the JavaDocs for `Region` you'll see that `Region.widthProperty()` returns `ReadOnlyDoubleProperty`, because they are using `ReadOnlyIntegerWrapper` internally and want `widthProperty()` to return a truly "read only" object.  This pattern is repeated frequently through the JavaFX library.

I think that most programmers are used to dealing with `ReadOnly{Type}Property` as return values from the JavaFX API, and probably don't even know about `{Type}Expression`, even though these two class categories differ only in support for the Java Bean methods.  

At the end of the day, `ReadOnlyDoubleProperty` does no harm.  Yes, you're giving/taking more than you need to, but that extra functionality is trivial and meaningless, so there's really no added coupling to worry about.  Nobody is ever going to call those two extra methods.

So, go with the flow and just use `ReadOnly{Type}Property` when you need something bind-able through the Fluent API

## Avoid Calling getValue()

Unless you want `Number` (and who does?), there's no advantage to calling `getValue()` on any `Observable` object.  Just use `get()` instead and you can largely ignore the differences between typed numeric `Obsevables` and the object versions.

# Conclusion

When I started out in JavaFX I used `IntegerProperty`, `DoubleProperty`, `StringProperty` and `BooleanProperty` virtually exclusively.  I never thought much about `ObjectProperty<T>` at all.  Eventually I became aware of `ObjectProperty<T>` and it wasn't clear to me how it was any better or worse than the typed `Properties`.

At some point I considered the typed `Properties` to be mostly convenience classes, that just made it a bit easier to declare stuff without all those angle brackets.  Later, I started to encounter occasional issues stemming from `IntegerProperty` implementing `Property<Number>` and not `Property<Integer>`.  At that point I started to view these as inconvenience classes.  I never had the same issues with `StringProperty` or `BooleanProperty`, though.

I'm not a big user of the Fluent API, and I went through a period when I didn't really use the typed `Properties` at all.  But there are still those times when the API forces you to contend with the typed `Properties` as most of the `Node Properties` are defined this way.

However, through years of changing my opinion and understanding about this, I've always used `StringProperty` and `BooleanProperty` over the generic `Properties`.  This is because you don't ever the encounter the issues created by `Property<Number>`.

Presently I lean towards just using the typed `Properties` unless there is a good reason not to.  Here's why:

* People are used to them, even if they don't understand them.
* The standard JavaFX `Nodes` are going to give them to you.
* 99% of the time, it just doesn't matter<br>You'll probably end up binding a `DoubleProperty` to a `DoubleProperty` anyway.
* It's trivial to convert when you need it.

The biggest remaining issue is defining parameters for your methods.  If you just need an `Observable` value to bind to something else without using the Fluent API, then you don't need anything more than `ObseravbleValue<T>`.  If you define the parameter as `ObservableValue<Double>` then it won't accept `DoubleProperty`.  Can you get away with `ObservableValue<Number>`?  If you can, then client code can pass you `DoubleProperty`.  But if you really, really to need deal with it as `ObservableValue<Double>`, then the client code can pass you `DoupbleProperty.asObject()`.
