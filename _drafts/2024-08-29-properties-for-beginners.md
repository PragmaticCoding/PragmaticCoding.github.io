---
title:  "An Introduction to Properties"
date:   2024-08-29 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/beginners-properties
ScreenSnap1: /assets/elements/ListObservables1.png
ScreenSnap2: /assets/elements/ListObservables2.png
ScreenSnap3: /assets/elements/ListObservables3.png
ScreenSnap4: /assets/elements/ListObservables4.png
Diagram: /assets/elements/ListProperties.png
part1: /javafx/elements/observable-classes-generics
part2: /javafx/elements/observable-classes-typed

excerpt: If you are a beginner with JavaFX and are wondering, "What are all these Property things anyway?", then this is the article for you.
---

# Introduction

One of the biggest advantages to JavaFX is it's huge library and integration for using the "Observer" Pattern.  At the same time, this can also be one of the biggest challenges to learning JavaFX, because the library is so extensive it can become overwhelming.  

In this article, we're not going to go too deep into the details.  This is intended for beginners so that they can get started and create their JavaFX applications the right way, and to avoid a lot of the pitfalls that many beginners fall victim to.

# What is a Property?

A `Property` is a special type of object designed to contain a data value and to support the "Observer" Pattern.  It does this by keeping track of whenever the value is changed, and running code that other `Property` objects that have registered with it when the value changes.  `Properties` can also "bind" to other `Properties`, so that when one or more of those other `Properties` changes, its value will be recalculated.

It's just about impossible to avoid `Properties` in JavaFX because virtually every aspect of every element of JavaFX is stored as a `Property` of some sort.  You can, to an extent, bury your head in the sand as just treat these as values that you can access through getters and setters, but sooner or later you're going to have to deal with the basic `Property` aspect of these values for some reason or another.

The exhaustive nature of the `Property` library in JavaFX and it's integration with all of the classes that make up JavaFX make a compelling argument that JavaFX is primarily designed to be used as a "Reactive" framework.  We'll go into this some more later, but for now we can just think of "Reactive" as what you get when you treat all of the `Properties` as `Properties`, construct your Presentation Model out of `Properties` and connect everything together through bindings.  

# Property Basics

There are a number of `Property` types, and we'll see them all in a bit.  For now, we'll just stick to `StringProperty`.  A `StringProperty` is a `Property` designed to hold a `String` (no big surprise there).  

At it's most basic, we can think of `StringProperty` as a container for a `String`.  We have a `get()` method that returns a `String` and a void `set(newVal : String)` method to update the value.  All of that is pretty straight-forward.

The most common thing you might do with a `StringProperty` is to bind it to another `StringProperty` using the `bind()` method.  This would look something like this:

``` kotlin
val x = SimpleStringProperty("abc")
val y = SimpleStringProperty("")
y.bind(x)
```
Now, any changes in `x` will be immediately reflected in `y`.  Also, any attempt to call the `y.set()` method will now generate a runtime error, indicating that a "bound value cannot be set".

But why would you want to do this?

Usually, one of those two `StringProperties`, either `x` or `y` is given to you from somewhere else.  In JavaFX it will probably be an internal `Property` of some JavaFX class.  Let's take a look at a more typical example:

``` kotlin
val x = SimpleStringProperty("abc")
val heading = Label()
heading.textProperty().bind(x)
```

Here we've replaced `y` with the `textProperty()` of a `Label`.  Usually, `x` would be an element of a Presentation Model, or it might be a `Property` variable that's global to your layout that you use to avoid coupling `Nodes` on your layout.  Either way, whenever `x` changes, the text on the screen associated with that `Label` will change.  

# Property-Like Objects

In JavaFX there are a number of types that behave very much like `Properties` but aren't actually `Properties`.  I think a lot of people use the term "Property" as a catch-all, meaning not just `Properties` but also these other types.  

So let's have a look at them...

## ObservableValue

Sometimes you don't want other code, or other people's code, updating the value in your `Property`.  Sometimes, you don't want to be given something that you can update.  In those cases, you can use `ObservableValue`.  You can do anything you can do with a `Property`, but you cannot call `set()` or `bind()` on an `ObservableValue`.

Here's an example:

``` kotlin
fun createLabel(obVal : ObservableStringValue)= Label().apply{
   textProperty().bind(obVal)
}
```
This is a function that returns a `Label` with its `textProperty()` bound to a supplied `Property`, but, since it isn't going to update the value in that `Property` it just asks for an `ObservableStringProperty`.  

## ReadOnlyProperty

This is something that you'll see much more than you will ever create.  In fact, you'll probably never create a `ReadOnlyProperty`.  There are lots of `Nodes` in JavaFX that have `Properties` that they really, really don't want you to change.  Internally, they will manipulate the value, but they are only will to give you something that you can never change.  That something is `ReadOnlyProperty`.

For instance, `Region` has a `widthProperty()` function that returns a `ReadOnlyDoubleProperty`.  You'll find a `Region.getWidth()` function, but you won't find a `Region.setWidth()` function.  Region has things that you can do to influence how it calculates its width, but you cannot directly set it.  So `Region.widthProperty()` doesn't give you something that will let you change the width directly.  

For most purposes you might have, you can treat `ReadOnlyProperty` exactly like `ObservableValue`.

## Binding

OK, `Binding` is a type as well as an action/method.  You will need to get these straight in your head.

For the sake of clarity, I will always use `Binding` for the type, and binding for the action.  

A `Binding` is an object that recalculates its value whenever any of the `Properties` that it is dependent upon changes.  Every `Binding`, no matter how you create it has a method called `computeValue()`.  This `computeValue()` method will return a value that is of the type that the `Binding` is declared as.  So `StringBinding.computeValue()` will return a `String`.

Every `Binding` when it is created, will have a list of `Properties` that it is bound to, which are called its "dependencies".  Whenever any one of its dependencies changes, the next call to the `Binding.get()` will call `Binding.computeValue()` and `Binding.get()` will return that new value.  

It's important to note that the dependencies are simple used to trigger a recalculation of the value.  There is no rule that says that `Binding.computeValue()` has to use any of these dependencies in its calculation, or that it cannot use other values that aren't listed as dependencies.  That being said, 99% of the time, the dependencies will be the values that are use in `Binding.computeValue()`.

## ObservableLists

An `ObservableList` is very much like a `Property` wrapping a `List` of some sort.  The key word there is "like".  `ObservableLists` are quite a bit different from the other `Properties` that we discuss here.

Instead of reporting on changes to a value, it reports changes to a list of values.  These changes include additions to the list, removals from the list and replacement of items in the list.

# Invalidation and Listeners

When you are talking about `Properties` and `bind()`, it's all about "changes".  But there's actually more going on than just that.  There is a very, very important concept called "Invalidation".

Simple put, `Properties` need a mechanism to tell other properties that they need to come and fetch the latest value that they hold, because the one that they last read may not be correct any more.  This mechanism is called "Invalidation".  Whenever you call `Property.set()`, that will cause an internal `valid` flag inside the `Property` to flip to `false`.  The only way to flip the `valid` flag back to `true` is to call `Property.get()`.

So how do the other `Properties` know that our `Property` has been invalidated?

They know because they register `Listeners` with our `Property`.  A listener is just a snippet of code that's called by the `Property` that just invalidated and is donated by the listening object.  Note that a `Listener` can be created by any code, and doesn't need to be part of a `Property` itself.

Inside the `Property` it keeps a list of all of the `Listeners` that have been registered with it.  When it is invalidated, the `Property` will then execute each of those `Listeners`, one after the other.

But here's one of the most important things:  If a `Property` is already invalid, then it cannot be invalidated again and it will not execute any of those `Listeners` until after it has been revalidated.  

## If a Tree Falls in the Woods...

...and there's nobody to here it.  Does it make a sound?

If a `Property` gets invalidated and there's no `Listeners` to notify, does anything happen?

I don't know about the tree, but a `Property` with no `Listeners` is pretty much just an object with a value field.  It will get revalidated if some code somewhere calls its `get()` method, but that won't make a difference to anything either.

## Invalidation Chains

When a `Property` is bound to another `Property` it registers a special kind of `Listener` with that `Property` that simply invalidates the second `Property` too.  Then, if that `Property` is, in turn, bound to another `Property` then its `Listener` will simply invalidate the third `Property`.  And so on, and so on, and so on.

Normally, at some point at the end of the chain is a `Listener` that actually calls `get()` on the `Property` it's listening to, and that will trigger the binding logic to call `get()` on the next `Property` back up the chain.  All the way back to the beginning.  And then coming back down the chain, all of those `Properties` will be revalidated.  

These chains are why an invalidate `Property` only might possibly have a different new value.  And why invalidation doesn't mean that a value *has* changed.  Let's look at an example:

``` kotlin
val x = SimpleIntegerProperty(3)
val y = SimpleBooleanProperty(false)
y.bind(x.greaterThan(7))
x.set(6)
```
At the end of this snippet of code, `x` has been invalidated, and then, because of the binding, so has `y`.  However, `y` started out `false`, and then was still `false` after it was bound.  We haven't looked at this stuff yet, but `IntegerProperty.greaterThan()` creates a `Binding`, which `y` is then bound to.  That `Binding` is invalidated as soon as its dependency, `x`, is invalidated.  Then, in turn `y` becomes invalidated.  However, `y` still has the same value, `false`.

## ChangeListeners

`ChangeListeners` are different from `InvalidationListeners` in two ways:

1. The trigger only when the value actually has changed.
1. They receive both the old value and the new value in the `Obseravble`.

# Instantiating Properties

If you want to create a brand new place to store a value that you want to be observable, you're going to need to instantiate a `Property` of some sort.  There are a number of different types of `Properties`, and you'll need to pick the right one.  Notably, you'll probably need to pick from one of these:

1. StringProperty
1. BooleanProperty
1. IntegerProperty
1. DoubleProperty
1. ObjectProperty

`ObjectProperty` is used for pretty much anything that isn't a `String`, `Boolean`, `Integer` or `Double`.  This could be a `LocalDate`, `BigDecimal`, an `Enum` or any other custom class you have created.  

But you'll find out pretty quickly that you can't instantiate any of these classes because they are all abstract.  But each one of these classes has a concrete subclass that starts with "Simple".  For example, `SimpleStringProperty`.  Instantiate using these classes, but type your variables as the super-class.

Like this:

``` kotlin
val nameProperty : StringProperty = SimpleStringProperty("Fred")
```
Notice that I've used `val` here and not `var`.  This is the same as using `final` in Java.  It's a good practice to establish all of your `Properties` as `final`.  The value inside them can change, even if they are `final`, but it will probably break everything if something changes the variable reference for the `Property` itself.

# Bindings

There are three ways to create a `Binding`:

1. The "Fluent API"
1. Static methods in the `Bindings` class
1. The "Low Level API" (creating a custom class)

Inside the JavaFX library, the Fluent API works by calling methods in the `Bindings` class which, in turn, creates custom `Binding` classes on the fly.  So you can see that everything eventually boils down to custom `Binding` classes.  

Because of that, we'll look at them in reverse order:

## Custom Binding Class

Let's look at a simple example:

``` kotlin
class NameBinding(private val fName: ObservableValue<String>,
                  private val lName :ObservableValue<String>) : StringBinding() {
   init {
     super.bind(fName, lName)
   }

   override fun computeValue() : String {
      return "${lName.get()}, ${fName.get()}"
   }
}
```
The first thing to note is that we are extending `StringBinding`.  Just like with `Properties`, we have about 5 different common types of `Bindings` to hold different kinds of values.  

The next thing to note is that we are accepting `ObservableValue<String>` as our dependencies.  This means that we are accepting the widest possible range of `Observable` types that we possibly can.  This will work with `Property<String>`, `StringProperty`, `StringBinding` and `ObservableStringValue`, which keeps our `Binding` as flexible as possible.  

The `init{}` section in Kotlin is very much like a constructor in Java.  This `init{}` section establishes the two passed parameters as dependencies for the binding by passing them to `super.bind()`.  This is super important.

Finally, we have `computeValue()`.  This is the method that's called to actually figure out what the `Binding's` value is.  Here we are just combining them together with a ", " between them.  Note that this method returns `String` not an `Observable` wrapping `String`.

Of course, you can also create something very similar as an anonymous inner class, if you like.

## The Bindings Class

From the JavaDocs:

> Bindings is a helper class with a lot of utility functions to create simple bindings.

That's an understatement.  There are literally hundreds of static methods in this class, and they all build `Bindings`.

If we wanted to do the same thing as our custom `Binding` example, here's how we would do it with `Bindings`:

``` kotlin
val binding = Bindings.createStringBinding({"${lName.get()}, ${fName.get()}"}, fName, lName)
```
This is the definition of the method:

``` java
public static StringBinding createStringBinding(Callable<String> func, Observable... dependencies)
```
In this case, it might be easier to think of `Callable<String>` as a `Supplier<String>`.  It's a function that takes no input parameters and returns a `String`.

It's easy to see how this maps to our custom `Binding`.  The first parameter corresponds to whatever code that we would put into `computeValue()` and the other parameters are the same dependencies that we passed to `super.bind()`.

You should also know that `Bindings` has a method called `concat()` which is specifically designed to do this kind of `Binding`.  We could use it instead:

``` kotlin
val binding = Bindings.concat(lName, ", ", fName)
```
It's clearly easier to use this method.

## The Fluent API

The Fluent API provides as set of methods for `Observables` that allow you to transform, combine, compare and do various operations on the `Observables`, returning new `Observables` that are bound to the original `Observable`.  That sounds like a mouthful, but it allows you to do things like this:

``` kotlin
val binding = integerProperty.add(2)
```
Whenever `integerProperty` changes, `binding` will be updated to have a value 2 greater than `integerProperty`.

Keeping with our name combination example, we can do this:

``` kotlin
val binding = lName.concat(", ").concat(fName)
```
