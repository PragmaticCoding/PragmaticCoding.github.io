---
title:  "Guide To the Observable Classes - Part II"
date:   2024-08-03 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/observable-classes-typed
ScreenSnap1: /assets/elements/QuickMVCI_1.png
Diagram2: /assets/elements/StringProperties.png
Diagram3: /assets/elements/IntegerProperties.png
Subclasses: /assets/elements/Properties2.png
ScreenSnap3: /assets/posts/StarterFX.png
part1: /javafx/elements/observable-classes-generics
wrapper: /javafx/elements/observable-classes-generics#readonlyobjectpropertybase-and-readonlyobjectwrapper

excerpt: Confused by the plethora of Property, Binding and Observable classes?  This guide will tell you what you really need to know about the typed Observable classes.
---

# Introduction

In [Part 1]({{page.part1}}) of this series, we looked at the `Observable` classes designed to wrap aribitrary `Object` values.  In this article we are going to move on to the typed `Observable` classes and interfaces.  We'll look at how they are different from the generic types, and how to deal with compatibility issues between the two.

If you haven't read [Part 1]({{page.part1}}) of this series, you should probably go and do that now, as the information in this article will make more sense to you.


# The Typed Classes and Interfaces

There are 6 sets of typed `Observable` classes and interfaces.  The each deal with one of the following types:

1. String
1. Boolean
1. Integer
1. Double
1. Long
1. Float

We don't need to look at all of them because they break down into two groups: The numeric types, and the non-numeric types.  We will look at just one type from each group, `String` for the non-numeric and `Integer` for the numeric.  The non-numeric is simpler, so we'll start with that:

# String Observables

The `String` types are a little bit more interesting to look at than `Boolean` because the Fluent API library for `String` is larger.  So we'll use `String` as our example type for a deep dive.  Here's the chart:

![String Properties]({{page.Diagram2}})

Here you can see that all of the generic `<T>`'s have been replaced with `<String>`.  Understand, though, that there is no interface called `ObservableValue<String>`, it's just `ObservableValue<T>` with the `<T>` resolved as a `<String>`.  The same holds true for all of the other entries on the chart defined with `<String>`.

## The Interfaces

We'll start with the interfaces.  Some of the descriptions are a little bit stripped down from what you'll see in [Part 1]({{page.part1}}).  I want this article to be complete within itself, but I don't want to repeat a lot of information that you can find in [Part 1]({{page.part1}}).

### Observable & ObservableValue\<String\>

At the top of the chart, you'll find the interface, `Observable`.  You'll notice that it is not typed at all, it's not even a generic.  That's because all of its methods do not deal with data.

This is the root for all of the other classes and interfaces, but it's really quite simple and only defines three methods: `addListener()`, `removeListener()` and `subscribe()`.  These are all related to the process called "Invalidation".

The next interface down is `ObservableValue<String>`, which extends `Observable`.  The key method in this interface is `ObservableValue.getValue()`, which allows us to actually read the value and will re-validate the `Observable`.

Now that we can read the value we can have the methods related to `ChangeListeners`, including those related to `Subscriptions`.  The other methods it adds are the `map()` and `flatMap()` methods used for transforming an `ObservableValue` into a different `ObservableValue`.

There is also the `ObservableObjectValue<String>` interface, which introduces the `get()` method, which at this point is effectively identical to `getValue()`.

### ObservableStringValue

`ObservableStringValue` adds no new methods and simply extends `ObservableObjectValue<String>`.  Essentially, you can use these two interfaces interchangeably.

However, if you are writing code that deals with the specific `String Observables` using `ObservableStringValue` might give a more consistent feel to the code.

### ReadOnlyProperty\<String\> & WritableValue\<String\>

These are the two parent interfaces for all of the `Property` classes.  

`ReadOnlyProperty<String>` simply introduces the `getBean()` and `getName()` methods.  These are presumably meaningful methods if you are serializing your `Properties`.

The interface `WritableValue<String>` gives us the `setValue()` method.  Now we can actually put something into an `Observable`!

### WritableObjectValue\<String\> and WritableStringValue

`WritabaleStringValue` is an extension of `WritableObjectValue<String>` that adds no methods.  So you can treat it as a synonym for `WritableObjectValue<String>`.

These two classes add the `set()` method, which, for `String` types is identical to `setValue()`.

### Property\<String\>

The `Property` interface extends the previous two interfaces and adds the ability to bind to other observables.  We get `bind()` and `bindBidirectional()`, and their corresponding unbinding methods.  Also there is the `isBound()` method that will tell us if the `Property` has been bound to something.

### Binding\<String\>

`Binding<String>` is the last interface on the chart.  It extends`ObservableValue<String>` adding a method to force the `Binding` to invalidate, a method to check if the `Binding` is valid and a method to get the dependencies of the `Binding`.

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

# Number Types

This is the most confusing part of the entire discussion.  JavaFX have utilizes the standard Java class called `Number` to create a framework for all of the numeric `Observables`.  I'm not sure what this adds, other than a layer of complexity, but it's a layer that you can't ignore.  

The `Number` types of `Observables` share many of the same differences from the generic `Observables` as the `String` types do.  So in this section we're going to concentrate on the extra complexity that comes from using `Number` for all of these variants.

All four of the numeric types work the same way.  Let's look at the chart for `Integer`:

![Integer Chart]({{page.Diagram3}})

## The Generically Defined Types

The generic classes in this chart all resolve to `<Number>`, not `<Integer>`.  So `IntegerProperty` implements `Property<Number>`, not `Property<Integer>`.  This means that if you have an instance of `ObjectProperty<Integer>` it is not type compatible with `IntegerProperty`.  This can be a source of frustration.

## ObservableNumberValue && ObservableIntegerValue

The `ObservableNumberValue` interface extends `ObservableValue<Number>`, which gives it the `Listener`, mapping and `Subscription` methods.  Its `getValue()` method returns `Number`.

Additionally, it has type specific `getValue()` equivalents: `doubleValue()`, `floatValue()`, `intValue()` and `longValue()`.  

`ObservableIntegerValue` extends `ObservableNumberValue()` and adds one method, `get()`.  However `ObservableIntegerValue.get()` returns `int` not `Number`.

This is very significant.

This is the first example that we've seen where `get()` returns a different value from `getValue()`.  This suggests to me, that when you are retrieving the value from a numeric `Observable`, you can largely ignore their `<Number>` nature if you just use `get()` all the time instead `getValue()`.

This is actually annoying with Kotlin because Kotlin creates "synthetic accessors" for Java fields that follow the Java Bean structure.  So this:

Also, `ObservableBooleanValue.get()` returns `boolean` instead of `Boolean` returned from `ObseravbleBooleanValue.getValue()`.  It's a smaller change, but still a difference.

## WritableNumberValue & WritableIntegerValue

`WritableNumberValue` is what the JavaDocs describe as a "tagging interface".  It has no methods defined, and simply appears to be a common ancestor for the numeric `Writable{Type}Value` interfaces.

`WritableIntegerValue` is a parallel to `ObservableIntegerValue` but for writing.  We have a difference between the two setters, as `WritableIntegerValue.set()` takes an `int`, while the `setValue()` method takes `Number`.

## NumberExpression

Unlike `ObjectExpression` and `StringExpression`, `NumberExpression` is an interface, not a class.  It defines methods for the four basic math operations - add, subtract, multiply and divide - and these operations all return a `NumberBinding`.  Additionally, it defines a number of comparison methods, all of which return `BooleanBinding`.  There's also `asString()` which returns a `StringBinding`.

For the math operations, each method is overloaded to take 5 possible parameter types.  The first four are non-`Obseravble` number classes: `double`, `int`,`long` and `float`.  The fifth is for `ObservableNumberValue`.  Regardless of the parameter type, you always get a `NumberBinding` (which we haven't seen yet) in return.

## NumberExpressionBase

This is our first class on the binding side of the chart, and this is abstract and partially implements `NumberExpression`.

This class has concrete implementations of all of the comparison methods, and for the numeric operation methods taking `ObservableNumberValue` as a parameter.  In other words, you'll find `add(ObservableNumberValue other)`, but not `add(int other)`, or `add(double other)`.  

## IntegerExpression

Another abstract class, `IntegerExpression` supplies all of the concrete implementations of the methods defined in `NumberExpression` that are not implemented in `NumberExpressionBase`.  However, it returns values that match the type of the result of the operation.  

This means that `IntegerExpression.add(int other)` will return a `IntegerBinding` instead of a `NumberBinding`.  

Also, this class has `asObject()` which returns a linked `ObjectExpression<Integer>`.  More on this later.

## NumberBinding  & IntegerBinding

`NumberBinding` is another "tagging" interface.  It has no methods, but extends `Binding<Number>` and `NumberExpression`.  

`IntegerBinding`, just like `StringBinding` is an abstract class that provides the starting point for any custom `Bindings` that you want to write, but for `Integer` values.  Note though, that it implements `Binding<Number>` and `ObservableValue<Number>`, not `Binding<Integer>` and `ObservableValue<Integer>`.

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
