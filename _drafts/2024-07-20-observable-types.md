---
title:  "Guide To the Observable Classes"
date:   2024-07-09 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/observable-classes
ScreenSnap1: /assets/elements/QuickMVCI_1.png
Diagram: /assets/elements/Properties.png
Diagram2: /assets/elements/StringProperties.png
Diagram3: /assets/elements/IntegerProperties.png
Subclasses: /assets/elements/Properties2.png
ScreenSnap3: /assets/posts/StarterFX.png

excerpt: Confused by the plethora of Property, Binding and Observable classes?  This guide will tell you what you really need to know.
---

# Introduction
If you surf over to the JavaDoc page for [Observable](https://openjfx.io/javadoc/21/javafx.base/javafx/beans/Observable.html), you'll see lists of subinterfaces and implementing classes that looks like this:

![All the Subclasses]({{page.Subclasses}})

That really does look complicated, and in this article we're going to break it down and put some structure to it so that you can understand exactly how you should be using the various classes and interfaces in a thoughtful and logic manner in your applications.

The things that we're not going to look at here are the `List`, `Map` and `Set` based interfaces and classes.  Those are a topic big enough for their own article.


# The Main Classes and Interfaces

Even though that huge list of interfaces and classes look daunting, there are lots of entries which a created to wrap specific data types.  If we strip those away, and just concentrate on the classes and interfaces that are defined generically, we can create a diagram like this:

![Diagram]({{page.Diagram}})

In order to understand how this all goes together, it's best to look first at the interfaces, as they tell us *what* the functionality is, and then look at the classes because they tell us *how* the functionality is implemented.

So we'll start with the interfaces...

## The Interfaces

### Starting at the Top: Observable

At the top of the chart, you'll find the interface, `Observable`.  This is the root for all of the other classes and interfaces, but it's really quite simple and only defines three methods: `addListener()`, `removeListener()` and `subscribe()`.  These are all related to the process called "Invalidation".

Invalidation is **the key concept** with `Obseravbles`.  When an `Observable` has become "invalidated" it means that *might* have changed its value, fires an `InvalidationEvent` and all of the objects that have registered `InvalidationListeners` with the `Observable` have their `InvalidationListeners` triggered.  It's up to these other classes to recheck the `value` of the `Observable` inside their `Listeners`.

It's possible for an `Observable` to be invalidated without having its value change.  Consider this code:

``` kotlin
val numberProperty = SimpleIntegerProperty(2)
val booleanObservable = numberProperty.greaterThan(5)
numberProperty.setValue(4)
```
After the third line of code runs, `booleanObservable` will have been invalidated, but its value still remains unchanged as `false`.  

One important point about invalidation is that `Observables` remain invalidated until the value is read, at which point they are validated again.  An `Observable` that has been invalidated cannot be invalidated again until after it has been revalidated.  At this point, we don't have any methods defined that will do that...

### ObservableValue

The next interface down is `ObservableValue`, which extends `Observable`.  This interface adds methods related to `ChangeListeners`, including related `Subscriptions`.  The other methods it adds are the `map()` and `flatMap()` methods used for transforming an `ObservableValue` into a different `ObservableValue`.

Between these two interfaces, we get the core functionality for observability: invalidation and changes.  Changes, of course, are based on invalidation, and every `ChangeListener` has, at the heart of its implementation, an `InvalidationListener`.  

Additionally, we also have `ObservableValue.getValue()`, which allows us to actually read the value and will re-validate the `Observable`.

There is also the `ObservableObjectValue<T>` interface, which introduces the `get()` method, which at this point is effectively identical to `getValue()`.

### ReadOnlyProperty and WritableValue

These are the two parent interfaces for all of the `Property` classes.  

`ReadOnlyProperty` is, in my opinion, a useless and annoying interface because it simply introduces the `getBean()` and `getName()` methods.  These are presumably meaningful methods if you are serializing your `Properties`, but honestly, who's going to do this?  `Properties` are, by their nature GUI elements and I'm not sure how you'd ever need Java Beans for them.  

However, because of this, all of our concrete `Properties` will need to implement these two methods.

The interface `WritableValue` gives us the `setValue()` method.  Now we can actually put something into an `Observable`!

This, by the way, is what makes a `Property` a `Property` and not just an `Observable` or `Binding`.  The ability to call `setValue()`.


### Property

The `Property` interface extends the previous two interfaces and adds the ability to bind to other observables.  We get `bind()`, `bindBidirectional()` and their corresponding unbinding methods.  Also there is the `isBound()` method that will tell us if the `Property` has been bound to something.

You can see now why binding has to come below `WritableValue`.  Whatever mechanism underlies a binding function will need to call `setValue()` to work.

### Binding

`Binding` is the last interface on the chart.  It doesn't seem to add much to `Observable` and `ObservableValue` except a method to force the `Binding` to invalidate, a method to check if the `Binding` is valid and a method to get the dependencies of the `Binding`.

These are all key methods, however.  Since a `Binding` is the key way to connect `Observables` it really works through controlling validation.  These methods give the tools to work with `Bindings`.

## The Classes

### ObjectExpression

This is the root class for all of the `Property` classes, even though the diagram goes diagonally down to `ObjectProperty`.

This class provides the core functionality for the "Fluent API" for creating bindings.  For `ObjectExpression` we have versions of `asString()`, each of which provides a `StringBinding`.  We also have methods for equality, inequality and checking for a `Null` value, each of which creates a `BooleanBinding`.

This is the only class in the chart that introduces new concrete methods that don't implement methods already defined by any of the interfaces in the chart.

### ReadOnlyObjectProperty

This class extends `ObjectExpression` and implements `ReadOnlyProperty`.  As we've seen, there's only two methods in `ReadOnlyProperty` and they aren't much practical use.  `ReadOnlyObjectProperty` doesn't provide any implementation for these methods either.  There really isn't much of a reason to pass an `Observable` around as a `ReadOnlyObjectProperty` more than an `ObjectExpression`, but you tend to see `ReadOnlyObjectProperty` used more often.

`ReadOnlyObjectProperty` can also be thought of as very much like `ReadOnlyProperty` but with the ability to use the Fluent API with it to create bindings.

### ObjectProperty

The best way to think about `ObjectProperty` is an implementation of `Property` but, since it inherits from `ObjectExpression` through `ReadOnlyObjectProperty`, also has the methods for creating bindings via the Fluent API.  However, it only has implementations for bidirectional binding and setting the value.  And it's abstract, so you cannot use it directly without extending it and supplying a ton of functionality.

This is, however, a great class to pass your already instantiated `Properties` around as, or to declare your variables as.  Like this:

``` kotlin
val someProperty : ObjectProperty<ClassA> = SimpleObjectProperty(someClassA)
```


### ObjectPropertyBase

This is the first truly useful abstract class for `Properties`.  This class adds *almost* all the remaining methods defined in the various interfaces up hierarchy.  We get methods to bind and unbind, add and remove `Listeners` and a `get()` method.  

The methods that are missing are the two silly Java Bean related methods.  This is the class that you probably want to extend from if you want to create your own `Property` classes, especially if you want to ignore Java Bean stuff as much you can.

### SimpleObjectProperty

This is the class that everyone is used to using, as it's the only `Property` class in the chart that isn't abstract.  

What makes `SimpleObjectProperty` different from `ObjectPropertyBase`?

It adds the `getBean()` and `getName()` methods and the infrastructure and constructors to set their values.  That's it.  You're probably never going to use those two methods either, nor are you going to use the constructors to set their values.  So you can treat this is pretty much equivalent to `ObjectPropertyBase` 99% of the time.

### ObjectBinding

This is the sole class on the `Binding` side of the chart, and it's abstract and it extends `ObjectExpression`.  It implements all of the methods defined in `Observable` and `ObservableValue` and inherits those in `ObjectExpression`.  

In addition, there are a few protected methods designed to make it easier to create custom classes based on `ObjectBinding`.  We get `bind()` and `unbind()` to start with. We also get `allowValidation()`, `onInvalidating()` and `isObserved()`.  You are pretty much required to use `bind()` in any custom class that you extend from `ObjectBinding` or else your binding is not going to be much use.

There is one abstract method, and that's `computeValue()` which is also protected.  

The intention is clear about the standard use case for custom classes extended from `ObjectBinding`.  It's expected that you'll either pass the `Observables` to be bound in the constructor (or use values available in the scope in which your class is defined) and bind them in the constructor of your class.  You'll also define a `computeValue()` method that will use those bound values to determine the return value of any calls to `get()` or `getValue()`.

Note, however, that `computeValue()` is only called when a call is made to `get()` or `getValue()` when the `Binding` has been invalidated.  `ObjectBinding` has internal storage for the last computed value, and will just return that when a call to `get()` is made while the `Binding` is valid.

Since `ObjectBinding` extends from `ObjectExpression` you can use the Fluent API to modify the results or combine it with other `ObjectExpressions`.

### The Java Bean Classes

These are in the chart just for the sake of completeness.  You aren't going to use them.  I've never seen them used.  Just ignore them unless you really, really need to use them - in which case you can research them for yourself.

# The Typed Observable Classes

That chart is pretty simple, but what about the seemingly countless other classes and interfaces that are listed in the JavaDocs?

It turns out that all those extra classes and interfaces are, for the most part, related to the typed `Observable` classes.  We talking about these:
* `StringProperty`
* `BooleanProperty`
* `IntegerProperty`
* `DoubleProperty`
* `LongProperty`
* `FloatProperty`   

Along with all of the other interfaces and classes that need to be created to support them.

Let's start off by looking at one of the simplest examples, `StringProperty`...

## String Types

![String Properties]({{page.Diagram2}})

Here you can see that all of the generic `<T>`'s have been replaced with `<String>`.  Understand, though, that there is no interface called `ObservableValue<String>`, it's just `ObservableValue<T>` with the `<T>` resolved as a `<String>`.  

There's one important difference in the structure of this chart.  That's the inclusion `StringExpression`.  Let's talk about that...

### StringExpression

`StringExpression` is an abstract class that implements a number of `String` related operations such as `concat()` and `length()` in an `Observable` fashion.

This is the class that allows you to implement the "Fluent API" for `String` based `Observables`.  This is the class that lets you do things like this:

``` kotlin
  var label = Label()
  label.textProperty().bind(model.lastNameProperty().concat(", ").concat(model.firstNameProperty()))
```

This inclusion of `StringExpression` in the class hierarchy is what makes `ObjectProperty<String>` different from `StringProperty`.  The code snippet above assumes that `model.lastNameProperty()` is a `StringProperty`, not an `ObjectProperty<String>`.  Because the latter wouldn't work.  

In most other practical respects, `StringProperty` and `ObjectProperty<String>` are pretty much the same thing, and you can use them interchangeably.

### The Other Classes with "String" in Their Name

Besides `StringExpression`, we also have `ReadOnlyStringProperty`, `StringProperty`, `StringPropertyBase` and `SimpleStringProperty`.  These all differ from the corresponding generic object `Properties` in that they extend from `StringExpression` rather than `ObjectExpression`.  

For instance if you have a `StringProperty` then you can do this:

``` kotlin
val property: StringProperty = SimpleStringProperty("ABC")
val somethingElse = property.concat("DEF")
```
But you cannot do the same if `property` is `ObjectProperty<String>` because `ObjectExpression` does not define `concat()`.

## Number Types

This is the most confusing part of the entire discussion.  For some reason, the authors of JavaFX have decided to utilize the standard Java class called `Number` to create a framework for all of the numeric `Observables`.  I'm not sure what this adds, other than a layer of complexity, but it's a layer that you can't ignore.  

All four of the numeric types work the same way.  Let's look at the chart for Integer:

![Integer Chart]({{page.Diagram3}})

### The Generic Classes

The generic classes in this chart all resolve to `<Number>`, not `<Integer>`.  So `IntegerProperty` implements `Property<Number>`, not `Property<Integer>`.  This means that if you have an instance of `ObjectProperty<Integer>` it is not type compatible with `IntegerProperty`.  This can be a source of frustration.

### ObservableNumberValue

This class extends `ObseravbleValue<Number>`, which gives it the `Listener`, mapping and `Subscription` method.  Its `getValue()` method returns, `Number`.

Additionally it has type specific `getValue()` equivalents: `doubleValue()`, `floatValue()`, `intValue()` and `longValue()`.  

### ObservableIntegerValue

`ObservableIntegerValue` extends `ObservableNumberValue()` and adds one method, `get()`.  However `ObservableIntegerValue.get()` returns `int` not `Number`.

This is very significant.

This is the first example that we've seen where `get()` returns a different value from `getValue()`.  This suggests to me that you can largely ignore the `ObservableValue<Number>` nature of all of these numeric typed `Observables` if you just use `get()` all the time instead `getValue()`.

This is actually annoying with Kotlin because Kotlin creates "synthetic accessors" for Java fields that follow the Java Bean structure.  So this:

``` kotlin
val abc = integerProperty.value
```
is equivalent to:
``` kotlin
val abc = integerProperty.getValue()
```
meaning that in Kotlin, you're better off using:
``` kotlin
val abc = integerProperty.get()
```
Also, `ObservableBooleanValue.get()` returns `boolean` instead of `Boolean` returned from `ObseravbleBooleanValue.getValue()`.  It's a smaller change, but still a difference.

### WritableNumberValue

This is what the JavaDocs describe as a "tagging interface", whatever that means.  It has no methods defined, and simply appears to be a common ancestor for the numeric `Writable{Type}Value` interfaces.

### WritableIntegerValue

This is a parallel to `ObservableIntegerValue` but for writing.  We have a difference between the two setters, as `WritableIntegerValue.set()` takes an `int`, while the `setValue()` method takes `Number`.

### NumberExpression

Unlike `ObjectExpression` and `StringExpression`, `NumberExpression` is an interface, not a class.  It defines methods for the four basic math operations - add, subtract, multiply and divide - and these operations all return a `NumberBinding`.  Additionally, it defines a number of comparison methods, all of which return `BooleanBinding`.  There's also `asString()` which returns a `StringBinding`.

For the math operations, each method is overloaded to take 5 possible parameter types.  The first four are non-`Obseravble` number classes: `double`, `int`,`long` and `float`.  The fifth is for `ObservableNumberValue`.  Regardless of the parameter type, you always get a `NumberBinding` (which we haven't seen yet) in return.

### NumberExpressionBase

This is our first class on the binding side of the chart, and this is abstract and partially implements `NumberExpression`.

This class has concrete implementations of all of the comparison methods, and for the numeric operation methods taking `ObservableNumberValue` as a parameter.  In other words, you'll find `add(ObservableNumberValue other)`, but not `add(int other)`, or `add(double other)`.  

### IntegerExpression

Another abstract class, `IntegerExpression` supplies all of the concrete implementations of the methods defined in `NumberExpression` that are not implemented in `NumberExpressionBase`.  However, it returns values that match the type of the result of the operation.  

This means that `IntegerExpression.add(int other)` will return a `IntegerBinding` instead of a `NumberBinding`.  

### NumberBinding

`NumberBinding` is another "tagging" interface.  It has no methods, but extends `Binding<Number>` and `NumberExpression`.  

### IntegerBinding



# Using this Information

## You Can't Instantiate ObservableValue (In Any Practical Sense)

If you look at the first chart, there's exactly two non-abstract classes: `SimpleObjectProperty<T>` and `ReadOnlyJavaBeanObjectProperty<T>`.  But let's be honest, the chances that you'll ever even figure out how to instantiate `ReadOnlyJavaBeanObjectProperty` are virtually zero (you need to instantiate a builder first), so we can just put that aside.  

Everything on the chart above `WritableValue<T>` cannot actually be implemented alone, because it won't have `setValue()`.  And if it doesn't have that, then it cannot be bound to anything nor can it ever change it's value - which would make it a very useless `Observable`.

This means that any time you get an `ObsevableValue<T>` that same object was instantiated somewhere as a `SimpleObjectProperty<T>` (or a custom equivalent) or is bound to something that was instantiated as `SimpleObjectProperty<T>`.

## Only Properties Can be Bound

Yes, `Binding<T>` is bound internally, but since it only implements `Observable` and `ObservableValue<T>` there is no external `bind()` method that you can call.

This means, that if you plan to bind an `Observable` to anything, you have to have it exposed to you as a `Property<T>` or an implementing class.  The same goes if you want to manually change the value through `setValue()`.  Although you could technically pass `WritableValue<T>` in that case, although you don't see that often.  Perhaps if you wanted someone to be able to set the value but not bind it, you could pass them `WritableValue<T>`.

## Method Overloading

In Java, you cannot overload methods where the signatures only differ in the types of enclosed generics.  For instance this code will fail to compile:

``` kotlin
fun doSomething(param1 : Property<String>) {
   .
   .
   .
}

fun doSomething(param1: Property<Int>) {
   .
   .
   .
}
```
You get an error that says something about "erase" and non-unique signatures.  What you can do instead is this:

``` kotlin
fun doSomething(param1 : StringProperty) {
   .
   .
   .
}

fun doSomething(param1: IntegerProperty) {
   .
   .
   .
}
```
This will work just fine.  This is the second reason (after the Fluent API) that you might want to use the typed `Observable` classes.

## Converting Between Generic and Typed Properties

As we've seen `IntegerProperty` implements `Property<Number>`, not `Property<Integer>`.  This means that it also implements `ObservableValue<Number>`.  

And that means that you cannot pass `IntegerProperty` as a parameter when `Property<Integer>` or `ObservableValue<Integer>` is required.  

This quickly becomes frustrating.

Fortunately all of the `Property<Number>` implementations have a function called `asObject()`.  For `IntegerProperty` this will pass you `Property<Integer>` that is bidirectionally bound to your `IntegerProperty`.  It's not the same object cast differently, but an actual new object that's bidirectionally bound to the original.

All of these classes also have a static method that does the reverse.  For instance, `DoubleProperty` has a method called `doubleProperty()`.  You pass it a `Property<Double>` and it gives you back a new `DoubleProperty` that is bidirectionally bound to the original `Property<Double>`.

Knowing about this makes any issues about whether or not to use typed or generic `Properties` a lot less critical.  You can very easily convert from one to the other when required.

## Dealing With Node Properties

You should be aware that virtually all of the JavaFX `Nodes` expose their `Properties` as typed `Properties` whenever possible.  For instance, `Region.widthProperty()` will return a `DoubleProperty` while, `Region.paddingProperty()` will return `ObjectProperty<Insets>`.

This makes sense, since, as a general purpose API, they would want to provide the most "out of the box" functionality, and returning `Properties` that can be used in the Fluent API without any conversion seems like the way to go.  You'll have to decide for yourself if this would influence the way that you declare your own `Properties`.

## When to Use ReadOnly{Type}Property

If you look at the JavaDocs for `Region` you'll see that `Region.widthProperty()` returns `ReadOnlyDoubleProperty`.  Now, we know that `ReadOnlyDoubleProperty` doesn't add any methods that aren't in `DoubleExpression` except for those two Java Bean methods.  

I can only conclude that they've returned `ReadOnlyDoubleProperty` in order to keep with the spirit of the API in providing support for serialization.  

But what should you return?  `ReadOnlyDoubleProperty` or `DoubleExpression`?  Which should you ask for as a method parameter?

From a purely technical perspective, you should never give or ask for more than you need to fulfil the intended purpose.  So `DoubleExpression` should win, right?

From a practical perspective, you're likely to confuse other programmers by being technically correct.  Everybody is used to getting `ReadOnlyDoubleProperty`, but they probably haven't even heard of `DoubleExpression` - and if they have, they probably don't understand what it does.

At the end of the day, `ReadOnlyDoubleProperty` does no harm.  Yes, you're giving/taking more than you need to, but that extra functionality is trivial and meaningless, so there's really no added coupling to worry about.  Nobody is ever going to call those two extra methods.

So, go with the flow and just use `ReadOnly{Type}Property` when you need something bind-able through the Fluent API

## Wrappers

# Conclusion

Given that just about any element on any of these charts that you encounter in real life was instantiated with or bound to something that is initialized as `SimpleSomethingProperty`, understanding these concepts in the charts is more about how you pass them around your application than anything else.  

The key idea is to limit coupling by exposing only as much functionality as you need to for any given purpose.  Essentially, this boils down to two things:

* How you pass `Observables` back as return values.<br>This speaks to how you expect, or want, client code to interact with the `Observables` that you pass back.  If you don't want the client code to be able to modify the value in the `Observable` then don't pass anything that implements `WritableValue`.  This means any kind of `Property` that doesn't have "ReadOnly" in its name.  
* How you define `Observables` as method parameters.<br>When you're writing a method, you know what you're going to do with the parameters that are passed to you.  If you are only going to use it as a dependency for a `Binding` then `ObservableValue` is generally good enough.  If you are going to use it as an element in a Fluent API binding, then use the appropriate `TypeExpression`, or `ReadOnlyTypeProperty`.  If you are going to change it's value in any way - including binding it to something - then you'll need to define your parameter as some sort of `Property`.  

## When to Use Generic vs Typed Properties

When I started out in JavaFX I used `IntegerProperty`, `DoubleProperty`, `StringProperty` and `BooleanProperty` virtually exclusively.  I never thought much about `ObjectProperty<T>` at all.  Eventually I became aware of `ObjectProperty<T>` and it wasn't clear to me how it was any better or worse than the typed `Properties`.

At some point I considered the typed `Properties` to be mostly convenience classes, that just made it a bit easier to declare stuff without all those angle brackets.  Later, I started to encounter occasional issues stemming from `IntegerProperty` implementing `Property<Number>` and not `Property<Integer>`.  At that point I started to view these as inconvenience classes.  I never had the same issues with `StringProperty` or `BooleanProperty`, though.

I'm not a big user of the Fluent API, and I went through a period when I didn't really use the typed `Properties` at all.  I think the Fluent API can be useful when the relationships are simple, but once things get complicated, then the Fluent API can get really hard to understand.  Since `ObservableValue.map()` has been added to the API, I see even less use for the Fluent API, except in those cases where you need to create a `Binding` depending two or more `Observables`.

There are times when the API forces you to contend with the typed `Properties` as most of the `Node Properties` are defined this way, and that can be a factor.

However, through years of changing my opinion and understanding about this, I've always used `StringProperty` and `BooleanProperty` over the generic `Properties`.  This is because you don't ever the encounter the issues created by `Property<Number>`.

Presently I lean towards just using the typed `Properties` unless there is a good reason not to.  Here's why:

* People are used to them, even if they don't understand them.
* The standard JavaFX `Nodes` are going to give them to you.
* 99% of the time, it just doesn't matter<br>You'll probably end up binding a `DoubleProperty` to a `DoubleProperty` anyway.
* It's trivial to convert when you need it.

The biggest remaining issue is defining parameters for your methods.  If you just need an `Observable` value to bind to something else without using the Fluent API, then you don't need anything more than `ObseravbleValue<T>`.  If you define the parameter as `ObservableValue<Double>` then it won't accept `DoubleProperty`.  Can you get away with `ObservableValue<Number>`?  If you can, then client code can pass you `DoubleProperty`.  But if you really, really to need deal with it as `ObservableValue<Double>`, then the client code can pass you `DoupbleProperty,asObject()`.
