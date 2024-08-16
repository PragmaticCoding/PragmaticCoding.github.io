---
title:  "Guide To the Observable Classes - Part I"
date:   2024-08-03 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/observable-classes-generics
Diagram: /assets/elements/Properties.png
Subclasses: /assets/elements/Properties2.png


excerpt: Confused by the plethora of Property, Binding and Observable classes?  This guide will tell you what you really need to know about the generic Observable classes.
---

# Introduction
One of the key elements to creating Reactive applications with JavaFX is understanding how the various `Obseravble` interfaces and classes work; how they relate to each other and how to use them best.

If you surf over to the JavaDoc page for [Observable](https://openjfx.io/javadoc/21/javafx.base/javafx/beans/Observable.html), you'll see lists of subinterfaces and implementing classes that looks like this:

![All the Subclasses]({{page.Subclasses}})

That really does look complicated, and in this article we're going to break it down and put some structure to it so that you can understand exactly how you should be using the various classes and interfaces in a thoughtful and logic manner in your applications.

# The Main Classes and Interfaces

Even though that huge list of interfaces and classes look daunting, there are lots of entries intended to wrap specific data types.  If we strip those away, and just concentrate on the classes and interfaces that are defined generically, we can create a diagram like this:

![Diagram]({{page.Diagram}})

In order to understand how this all goes together, it's best to look first at the interfaces, as they tell us *what* the functionality is, and then look at the classes because they tell us *how* the functionality is implemented.

So we'll start with the interfaces...

## The Interfaces

### Starting at the Top: Observable

At the top of the chart, you'll find the interface, `Observable`.  This is the root for all of the other classes and interfaces, but it's really quite simple and only defines three methods: `addListener()`, `removeListener()` and `subscribe()`.  These are all related to the process called "Invalidation".

Invalidation is **the key concept** with `Obseravbles`.  When an `Observable` has become "invalidated" it means that it *might* have changed its value, and all of the objects that have registered `InvalidationListeners` with the `Observable` have those `InvalidationListeners` triggered.  It's up to these other classes to recheck the `value` of the `Observable` inside their `Listeners`.

It's possible for an `Observable` to be invalidated without having its value change.  Consider this code:

``` kotlin
val numberProperty = SimpleIntegerProperty(2)
val booleanObservable = numberProperty.greaterThan(5)
numberProperty.setValue(4)
```
After the third line of code runs, `booleanObservable` will have been invalidated, but its value still remains unchanged as `false`.  

One important point about invalidation is that `Observables` remain invalidated until the value is read, at which point they are validated again.  

{% include notice type="warning" content = "An Observable that has been invalidated cannot be invalidated again until after it has been revalidated." %}

At this point, we don't have any methods defined that will read and revalidate an `Observable`...

### ObservableValue

The next interface down is `ObservableValue`, which extends `Observable`.  The key method in this interface is `ObservableValue.getValue()`, which allows us to actually read the value and will re-validate the `Observable`.

Now that we can read the value we can have the methods related to `ChangeListeners`, including those related to `Subscriptions`.  The other methods it adds are the `map()` and `flatMap()` methods used for transforming an `ObservableValue` into a different `ObservableValue`.

Between these two interfaces, we get the core functionality for observability: invalidation and changes.  Changes, of course, are based on invalidation, and every `ChangeListener` has, at the heart of its implementation, an `InvalidationListener`.  

There is also the `ObservableObjectValue<T>` interface, which introduces the `get()` method, which at this point is effectively identical to `getValue()`.

### ReadOnlyProperty and WritableValue

These are the two parent interfaces for all of the `Property` classes.  

`ReadOnlyProperty` is, in my opinion, a useless and annoying interface because it simply introduces the `getBean()` and `getName()` methods.  These are presumably meaningful methods if you are serializing your `Properties`, but honestly, who's going to do this?  `Properties` are, by their nature GUI elements and I'm not sure how you'd ever need Java Beans for them.  

However, because of this, all of our concrete `Properties` will need to implement these two methods.

The interface `WritableValue` gives us the `setValue()` method.  Now we can actually put something into an `Observable`!  There is also `WritableObjectValue` which adds the `set()` method, which is (at this point) identical to `setValue()`.

### Property

The `Property` interface brings together `WritableValue` and `ObservableValue` and adds the ability to bind to other observables.  We get `bind()` and `bindBidirectional()`, and their corresponding unbinding methods.  Also there is the `isBound()` method that will tell us if the `Property` has been bound to something.

You can see now why binding has to come below `WritableValue`.  Whatever mechanism underlies a binding function will need to call `setValue()` to work.

### Binding

`Binding` is the last interface on the chart.  It doesn't seem to add much to `Observable` and `ObservableValue` except a method to force the `Binding` to invalidate, a method to check if the `Binding` is valid and a method to get the dependencies of the `Binding`.

These are all key methods, however.  Since a `Binding` is the main way to connect `Observables` together, it really works through controlling validation.  These methods give the basic tools to construct custom `Bindings`.

## The Classes
Everything else on the chart is a class, let's take a look at them.

### ObjectExpression

This is a class you might not have heard about, but it is the root class for all of the `Property` classes, even though the diagram goes diagonally down to `ObjectProperty`.

This class provides the core functionality for the "Fluent API" for creating bindings.  In `ObjectExpression` we have several versions of `asString()`, each of which provides a `StringBinding`.  We also have methods for equality, inequality and checking for a `Null` value, each of which creates a `BooleanBinding`.  That's not really much, but when we look at the typed `Observable` classes in the Part II we'll see a lot more functionality with the Fluent API.

This is the only class in the chart that introduces new concrete methods that don't implement methods already defined by any of the interfaces in the chart.

{% include notice type="primary" content = "Every class that extends from ObjectExpression can use the Fluent API" %}

### ReadOnlyObjectProperty

This class extends `ObjectExpression` and implements `ReadOnlyProperty`.  As we've seen, there's only two methods in `ReadOnlyProperty` and they aren't much practical use.  `ReadOnlyObjectProperty` doesn't provide any implementation for these methods either.  There really isn't a practical reason to pass an `Observable` around as a `ReadOnlyObjectProperty` over an `ObjectExpression`, but you tend to see `ReadOnlyObjectProperty` used more often (we'll see why soon).

`ReadOnlyObjectProperty` can also be thought of as very much like `ReadOnlyProperty` but with the ability to use the Fluent API to create bindings.

### ObjectProperty

The best way to think about `ObjectProperty` is an implementation of `Property` but, since it inherits from `ObjectExpression` through `ReadOnlyObjectProperty`, also has the methods for creating bindings via the Fluent API.  However, it only has implementations for bidirectional binding and setting the value.  And it's abstract, so you cannot use it directly without extending it and supplying a ton of functionality.

This is, however, a great class to pass your already instantiated `Properties` around as, or to declare your variables as.  Like this:

``` kotlin
val someProperty : ObjectProperty<ClassA> = SimpleObjectProperty(someClassA)
```
This is a bit more versatile than passing around `Property<ClassA>`, because it will enable the Fluent API.

### ObjectPropertyBase

This is the first truly useful abstract class for `Properties` for extending to create custom classes.  This class implements *almost* all the remaining methods defined in the various interfaces up hierarchy.  We get methods to bind and unbind, add and remove `Listeners` and a `get()` method.  

The methods that are missing are the two silly Java Bean related methods.  This is the class that you probably want to extend from to create your own `Property` classes, especially if you want to ignore Java Bean stuff as much you can.  

If you look at the JavaFX source code, you'll see lots of examples of internally created custom `Properties` that are made by directly extending `ObjectPropertyBase`.

### SimpleObjectProperty

This is the class that everyone is familiar with instantiating, as it's the only "normal" `Property` class in the chart that isn't abstract.  

What makes `SimpleObjectProperty` different from `ObjectPropertyBase`?

It adds the `getBean()` and `getName()` methods and the infrastructure and constructors to set their values.  That's it.  You're probably never going to use those two methods either, nor are you going to use the constructors to set their values.  So you can treat this is pretty much equivalent to `ObjectPropertyBase` 99% of the time.

However, since this and `ObjectPropertyBase` do not *add* any methods to `ObjectProperty` other than the `SimpleObjectProperty` constructors, there's no reason to retain a reference to a `Property` as either `SimpleObjectProperty` or `ObjectPropertyBase`.  Consider these to be "implementation only" classes.

### ReadOnlyObjectPropertyBase and ReadOnlyObjectWrapper

`ReadOnlyObjectPropertyBase` is very similar to `ObjectPropertyBase` in that it is an abstract class that has almost all of the interface methods implemented except for the "Bean" methods.  It's also missing the `get()` method (you'll see why shortly).

But it also seems fairly useless.  What's the point of instantiating something as an `Obseravble` if the value can never change? The answer is in `ReadOnlyObjectWrapper`...

The `ReadOnlyObjectWrapper` class extends from `SimpleObjectProperty` and can be used exactly like `SimpleObjectProperty` any time you want.  However, it has one extra public method: `getReadOnlyProperty()` that returns a `ReadOnlyObjectProperty` that is synchronized with the `ReadOnlyObjectWrapper` that created it.

What's the point of this?

It all has to do with preventing client code from casting your "read only" `Observable` objects back to something implementing `WritableValue`.  Consider that the following code will run and print "Is SimpleObjectProperty: true":

``` kotlin
val x: ObservableValue<String> = SimpleObjectProperty("")
println("Is SimpleObjectProperty: ${x is SimpleObjectProperty}")
(x as ObjectProperty).set("abc")
```
The variable `x` was instantiated as `SimpleObjectProperty<String>` but declared as a reference to an `ObsevableValue<String>`.  The intention being that it should only be observable at this point.  However, it's still a `SimpleObjectProperty` and this cannot be hidden from any client code.  There's nothing stopping anyone from taking your `ObservableValue<String>` and casting it to `SimpleObjectProperty` or `ObjectProperty` and then calling its `set()` or `bind()` methods.  

How do you stop this?

`ReadOnlyObjectWrapper` to the rescue!

If you look at the source code for `ReadOnlyObjectWrapper` you'll find this:

``` java
public class ReadOnlyObjectWrapper<T> extends SimpleObjectProperty<T> {
    private ReadOnlyObjectWrapper<T>.ReadOnlyPropertyImpl readOnlyProperty;

    public ReadOnlyObjectProperty<T> getReadOnlyProperty() {
        if (this.readOnlyProperty == null) {
            this.readOnlyProperty = new ReadOnlyPropertyImpl();
        }

        return this.readOnlyProperty;
    }

    protected void fireValueChangedEvent() {
        super.fireValueChangedEvent();
        if (this.readOnlyProperty != null) {
            this.readOnlyProperty.fireValueChangedEvent();
        }

    }

    private class ReadOnlyPropertyImpl extends ReadOnlyObjectPropertyBase<T> {
        private ReadOnlyPropertyImpl() {
        }

        public T get() {
            return ReadOnlyObjectWrapper.this.get();
        }
    }
}
```
I've taken the "Bean" stuff out because it gets in the way.

There's the `ReadOnlyPropertyBase`!  And look, it delegates its `get()` method to the enclosing `ReadOnlyObjectWrapper` - and the value there can change.

You can see from this that `getReadOnlyProperty()` returns a completely separate `Property` object that extends `ReadOnlyObjectProperty` and has it's value connected back to what is essentially a `SimpleObjectProperty`.  And, as you can see from the charts, it does not implement `WritableValue` so it cannot be cast to any class that supports `set()`.

The following code will print "Is SimpleObjectProperty: false" and then fail with an exception when doing the cast:

``` kotlin
val x: ObservableValue<String> = ReadOnlyObjectWrapper("").readOnlyProperty
println("Is SimpleObjectProperty: ${x is SimpleObjectProperty}")
(x as ObjectProperty).set("abc")
```
But if we take off the call to `getReadOnlyProperty()` it will run just like the first example that allows the casting to work.  

This technique is used inside many of the JavaFX `Nodes`.  Let's look at the source code for `Region`:

``` java
public final ReadOnlyDoubleProperty widthProperty() {
    if (this.width == null) {
        this.width = new ReadOnlyDoubleWrapper(this._width) {
            protected void invalidated() {
                Region.this.widthChanged(this.get());
            }

            public Object getBean() {
                return Region.this;
            }

            public String getName() {
                return "width";
            }
        };
    }

    return this.width.getReadOnlyProperty();
}
```
Internally, the `width Property` is implemented as `ReadOnlyDoubleWrapper`, but `Region.widthProperty()` returns `width.getReadOnlyProperty()` which means that you can never cast it to something that supports `set()`.  However, inside of `Region` the `width Property` works just like any other read/write `Property`.

{% include notice type="primary" content = "Use ReadOnlyObjectWrapper and its getReadOnlyProperty() method when you really want to make sure that your client code cannot update your `Property` by casting." %}

### ObjectBinding

This is the sole class on the `Binding` side of the chart, and it's abstract and it extends `ObjectExpression`.  It implements all of the methods defined in `Observable` and `ObservableValue` and inherits those in `ObjectExpression`.  

In addition, there are a few protected methods designed to make it easier to create custom classes based on `ObjectBinding`.  We get `bind()` and `unbind()` to start with. We also get `allowValidation()`, `onInvalidating()` and `isObserved()`.  You are pretty much required to use `bind()` in any custom class that you extend from `ObjectBinding` or else your binding is not going to be much use.

There is one abstract method, and that's `computeValue()` which is also protected.  

The intention is clear about the standard use case for custom classes extended from `ObjectBinding`.  It's expected that you'll either pass the `Observables` to be bound in the constructor (or use values available in the scope in which your class is defined) and bind them in the constructor of your class.  You'll also define a `computeValue()` method that will use those bound values to determine the return value of any calls to `get()` or `getValue()`.

{% include notice type="warning" content = "Note that computeValue() is only called when a call is made to get() or getValue() when the Binding has been invalidated." %}

`ObjectBinding` has internal storage for the last computed value, and will just return that when a call to `get()` is made while the `Binding` is valid.

Since `ObjectBinding` extends from `ObjectExpression` you can use the Fluent API to modify the results or combine it with other `ObjectExpressions`.

We can look to the `Bindings` utility class for minimal implementation of `ObjectBinding`:

``` java
public static <T> ObjectBinding<T> createObjectBinding(final Callable<T> var0, final Observable... var1) {
    return new ObjectBinding<T>() {
        {
            this.bind(var1);
        }

        protected T computeValue() {
            try {
                return var0.call();
            } catch (Exception var2) {
                Logging.getLogger().warning("Exception while evaluating binding", var2);
                return null;
            }
        }

        public void dispose() {
            super.unbind(var1);
        }

        public ObservableList<?> getDependencies() {
            return (ObservableList)(var1 != null && var1.length != 0 ? (var1.length == 1 ? FXCollections.singletonObservableList(var1[0]) : new ImmutableObservableList(var1)) : FXCollections.emptyObservableList());
        }
    };
}
```
Most people don't bother with the logging, the `dispose()` method or `getDependencies()` when they create their own custom `Binding`, but the methods in `Bindings` are intended for a wide variety of applications.

### ReadOnlyJavaBeanObjectProperty

This is in the chart just for the sake of completeness.  You aren't going to use it.  I've never seen it used.  Just ignore it unless you really, really need to use it - in which case you can research it for yourself.


# Where the Methods are Defined

In reality, the `Observable` types don't define that many methods, and they can probably be best understand through a table:

| Type           | Methods          | Description |
| Observable     | addListener(InvalidationListener)<br>removeListener(InvalidationListener)<br>subscribe(Runnable)|Methods for InvalidationListeners|
| ObservableValue| getValue()<br>addListener(ChangeListener)<br>subscribe(Consumer)<br>subscribe(BiConsumer)<br>map() & flatMap()<br>when() | Methods for value based Listeners |
| ObservableObjectValue | get()  | |
| WritableValue | setValue() | The key method for updating and binding |
| WritableObjectValue | set() | |
| Property | bind()<br>bindBidirectional()<br>unBind()<br>unbindBidirectional()<br>isBound() | All the binding methods |
| ReadOnlyProperty | getBean()<br>getName() | The two Java Bean methods |
| ObjectExpression | asString()<br>isEqualTo() & isNotEqualTo()<br>isNull() & isNotNull() | The Fluent API for object `Properties` |
| Binding | getDependencies()<br>invalidate()<br>isValid() | The core functionality for a `Binding` |

All the other entries in the chart simply implement these methods, but don't introduce any new public methods.  So they do not appear in this table.


# Using this Information

I think that seeing all of this laid out in a charts and a table clarifies something that seems at first to be just mysterious and then later frustrating.  But what are the practical implications of knowing this?

## You Can't Instantiate ObservableValue

Despite all the complexity of the hierarchial chart,  any time that you encounter an `ObservableValue<T>` you know that it was instantiated in one of three ways:

1. As an extension of `ObjectPropertyBase<T>`, probably `SimpleObjectProperty<T>`
1. As an extension of  `ObjectBinding<T>`<br>This is true even if it was the result of using the Fluent API or the `Bindings` class library.
1. As an extension of `ReadOnlyPropertyBase<T>` from the `getReadOnlyProperty()` function of a `ReadOnlyObjectWrapper` instance.

This is true of any type on this chart that you receive.  

The point of this is that *how* you choose to pass `Observables` around in your application has more to do with what you want to expose about them than anything else.

Generally speaking, you want to pass a type as high up the chart as you can while still providing the functionality that you need.  This is going to become important in three places:

1. How you type variables
1. How you return values from methods
1. How you type method parameters

The first two pertain to how you expose information about your `Properties` to other code.  Never type a variable as `SimpleObjectProperty<T>`, use `ObjectProperty<T>` if you want to use the Fluent API, otherwise just use `Property<T>`.  And only use these two types if you want to be able to call `set()` or `bind()`.  

The last item in the list refers to limiting how much you know about `Properties` passed to you.  If all you are going to do with a `Property` make it the dependency of a `Binding`, then type it as `ObservableValue<T>`.  If you are going to use it in a Fluent API binding, then type it as `ReadOnlyObjectProperty`, or `ObjectExpression`.

What you are trying to do with all of this is to maximize versatility while minimizing risk.  Don't ask for something writable if you're not going to call `set()` or `bind()` on it.  Don't pass something writable if you don't want your client code to update it, pass a `ReadOnlyProperty` or an `ObservableValue` in those cases.

## Only Properties Can be Bound

Yes, `Binding<T>` is bound internally, but since it only implements `Observable` and `ObservableValue<T>` there is no external `bind()` method that you can call.

This means, that if you plan to bind an `Observable` to anything, you have to have it exposed to you as a `Property<T>` or an implementing class.  The same goes if you want to manually change the value through `setValue()`.  Although you could technically pass `WritableValue<T>` in that case, although you don't see that often.  Perhaps if you wanted someone to be able to set the value but not bind it, you could pass them `WritableValue<T>`.

# Conclusion

I hope that this article clears up most of the confusion that you might have had about what all these `Observable` types are all about and where they come from.  In the next article in this series, we are going to look at the typed `Observable` classes and interfaces.
