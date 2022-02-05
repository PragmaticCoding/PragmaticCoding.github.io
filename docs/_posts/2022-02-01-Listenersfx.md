---
title:  "Observable Listeners in JavaFX"
date:   2022-02-01 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/listeners
excerpt: ChangeListener or InvalidationListener, which is right kind of listener to use?    
---

If you are like me, when you started programming with JavaFX you found that `InvalidationListeners` are a mysterious and unfathomable thing.  It's not at all clear how they work, and why you would want to use them.  So you stay away, and use `ChangeListener` instead because it's pretty to understand what a "change" is, and how `ChangeListener` works.

But this is probably a mistake.  `InvalidationListener` is badly explained in the JavaDocs, but the concepts behind them are actually pretty simple and you should learn how to use them.  There's a good chance that `InvalidationListener` is a better fit for your needs that `ChangeListener`.

# Observable Values and Listeners

The concept of "observability" is key to any Reactive design, and in JavaFX it's baked in right from the start, but what does it mean?

In programming, the "Observer" pattern is a construct which allows observers to be registered with a value, and those observers will be notified when that value changes.  "Observable" in JavaFX turns that on it's head, and changes the focus from the observer to the observed.  It really just the same thing, but from a slightly different perspective.  

In JavaFX the Observer pattern is implemented by wrapping the data values into classes which which present methods to allow other classes to be notified when the enclosed value changes.  The notification mechanism is called a "Listener".  

## What Are Listeners For?

For the most part, listeners are used to trigger imperative code due to changes in the State of your application.  Most often, they are used to perform operations that need to be done off the FXAT, or to invoke functionality which simply isn't designed to be Reactive.  For instance, `Animations` are actions that need to be triggered, and this can be done through a listener.  In most cases, however, it's better to implement a more direct trigger like an `ActionEvent` if it's possible.

Sometimes a listener can be useful when it's too complicated to implement a `Binding`.  Perhaps, if there is an arbitrarily large number of elements that need to be connected in a complicated way, attempting to install a `Binding` on each one may be prohibitive.  In that case, having a single listener that then updates all of the dependent values might be less complicated and more clear.


Before we look at the listeners, let's take a look at how the data values are wrapped into observables.

# The JavaFX Observable Interfaces

In JavaFX, the great-great-grandfather of `Properties` and `Bindings` is the `Observable` interface.  The great-grandfather is the `ObservableValue` interface.  These two interfaces define the most important aspect of anything observable - that it can be listened to.

Let's have a quick look at both of these interfaces, and see what they do.

## Observable

`Observable` is a top level interface, so it doesn't inherit any methods from any other interfaces.  It has just two methods:

- addListener(InvalidationListener listener)
- removeListener(InvalidationListener listener)

That's it.  Pretty simple.  We'll look at what an `InvalidationListener` is in a little bit.

## ObservableValue

`ObservableValue` extends `Observable` and adds three new methods:

- addListener(ChangeListener<? super T> listener)
- removeListener(ChangeListener<? super T> listener)
- getValue()

Okay, so `ObservableValue` adds the ability to add and remove a `ChangeListener` onto a value, in addition to the `InvalidationListener` from `Observable`.  It also adds the ability to get the current value of whatever is inside the `ObservableValue`.  

You should also notice that `Observable` has no way to get the current value of it's object, which you'll see makes it not that useful by itself.

`ObservableValue`, though, is pretty useful if all you want is to catch when a value changes, grab the new value and do something with it.  Which, to be honest, is about 90% of the use cases for any observable class.

Remember that these are just interfaces, so while you can use them as a data type you can't actually instantiate them and for that you'll have to use some class that implements them.  Presumably, any class that you use to do this is also going to have some way to put data into the `ObservableValue` and change it.  We're not going to talk about that at all in this article.

# The Listeners

The two listeners are also interfaces, and they're both fairly simple.  Both of these are functional interfaces, so you'll most likely come across them defined as lambda statements.

## InvalidationListener

`InvalidationListener` has only one method:

- invalidated(Observable observable)

This method will be called whenever the value in the `Observable` becomes invalid, and it will pass the observed object itself to that method.  This is convenient if you are going to reuse the same listener for several `Observables`.

Okay, but what is it, and when is it called?

The JavaDocs are less than helpful:

> An InvalidationListener is notified whenever an Observable becomes invalid. It can be registered and unregistered with Observable.addListener(InvalidationListener) respectively Observable.removeListener(InvalidationListener)

It says to look at the JavaDocs for `ObservableValue`.  You'll find this:

> An ObservableValue generates two types of events: change events and invalidation events. A change event indicates that the value has changed. An invalidation event is generated, if the current value is not valid anymore.

And then it goes on about "lazy evaluation" without really explaining anything.

So now we "know" that invalidation events are triggered when the value becomes "invalid", and that change events are triggered when the value changes.  Let's look at `ChangeListener` before we try to figure out what "invalid" means.

## ChangeListener

`ChangeListener` also has only one method:

- changed(ObservableValue<? extends T> observable, T oldValue, T newValue)

This is a little bit more clear.  The method is called an it's passed the `ObservableValue` class itself, plus we get the old value and we get the new value.  That makes sense.

# Which Listener to Choose?

If you are like I was years ago, you got to that "becomes invalid" part in the JavaDocs and you said to yourself, "I don't get it.  `ChangeListener` is the way to go, because I understand that idea".  The Web wasn't much help either, because every site seemed to get caught up in details around lazy evaluation and "weak listeners".

So let's clear this up.

## What Does "Invalid" Mean?

Here are the things that define what "Invalid" is all about:

- First and foremost, "Invalid" is a state of the `Observable`.  The `Observable` is either "Valid" or "Invalid", and the `Observable` itself is going to keep track of this for us.

- The most important thing to understand is that the `Observable` is always set to "Valid" when  `getValue()` is called on it.  That's the only way to set it "Valid".

- The next most important thing to understand is that it can't became "more invalid".  If an `Observable` is already "Invalid" and something happens that would otherwise have turned it from "Valid" to "Invalid", nothing's going to change in the `Observable`.  It's already "Invalid".  

- Finally, an `Observable` is flipped to "Invalid" when it's possible that it's value *might* have changed.  

**Might?**

This is usually going to happen through `Bindings`.  In the example we're going to look at, we're going to have an `ObservableValue<Boolean>` that is based on whether or not an `ObservableValue<Integer>` is greater than 10.  If the initial value of the `ObservableValue` is 8 and then 1 is added to it, then your `ObservableValue<Boolean>` *might* have changed.  You're not going to know if it has unless you run the code for the `Binding`, which won't happen until you call `getValue()` on the your `ObservableValue<Boolean>`.

That's where `Invalidation` comes into play.  As soon as that 8 is changed to a 9, JavaFX is going to detect that your `Boolean` `ObservableValue` depends on it, and flag both of the `ObservableValues` as "Invalid".

In this case, calling `getValue()` on the `Boolean ObservableValue` is going to call it to call `getValue()` on the `Integer ObservableValue`, and both of them are going to get flipped back to "Valid" as a result.

### An Example

Let's take a look at how this works.  Here's a small program that puts a Button on the screen.  The program has two `Properties`, one is a `BooleanProperty` and the other is an `IntegerProperty`.  The `BooleanProperty` is bound to the `IntegerProperty` using the "Fluent" API, and it tests whether the `IntegerProperty`'s value is greater than 10.  The `IntegerProperty` starts off with a value of 7, and is incremented each time the `Button` is clicked.

There are two `InvalidationListeners`, one on each `Property`.  Each one simply does a `System.out.println()`, with some text that identifies it, and then it invokes the `getValue()` method on the `IntegerProperty` to show the current value of that Property.

``` java
public class InvalidationDemo extends Application {

    private final BooleanProperty overTen = new SimpleBooleanProperty(false);
    private final IntegerProperty counter = new SimpleIntegerProperty(7);

    public static void main(String[] args) {
        launch();
    }

    @Override
    public void start(Stage stage) throws IOException {
        overTen.bind(counter.greaterThan(10));
        overTen.addListener(ob -> System.out.println("Boolean invalid " + counter.getValue()));
        counter.addListener(ob -> System.out.println("Counter Invalid: " + counter.getValue()));
        Scene scene = new Scene(createContent(), 320, 240);
        stage.setScene(scene);
        stage.show();
    }

    private Region createContent() {
        Button button = new Button("Click Me!");
        button.setOnAction(evt -> {
            System.out.println("Button Clicked");
            counter.set(counter.getValue() + 1);
        });
        VBox vBox = new VBox(button);
        vBox.setPadding(new Insets(30));
        return vBox;
    }
}
```

The output looks like this:

```
Button Clicked
Boolean invalid 8
Counter Invalid: 8
Button Clicked
Counter Invalid: 9
Button Clicked
Counter Invalid: 10
Button Clicked
Counter Invalid: 11
```
What's going on here?

Notice that the InvalidationListener doesn't have any code that invokes `getValue()` on the `BooleanProperty`.  So the `BooleanProperty` is set to "Valid" when the `Binding` is established and there's no other code that will set it to "Valid" in this application.  At this point also, the `BooleanProperty` has a value of `false`.

After the first click on the `Button`, the `IntegerProperty` is set to "Invalid", and the `BooleanProperty` follows suit, as you would expect.  So both `InvalidationListeners` are triggered.  

This is important:  **Even though the BooleanProperty is now "Invalid", if it were to be recalculated it would remain "false".**

Invalidation simply means that it needs to be checked to see what it's current value is.  And if you don't check it, it remains "Invalid".

Both of these `InvalidationListeners` invoke `getValue()` on the `IntegerProperty`.  So it's status is set back to "Valid".  But the status of the `BooleanProperty` remains "Invalid", since its `getValue()` is never called.

On to click number 2.  We see the click, and then we see the `InvalidationListener` called for the `IntegerProperty`, but not for the `BooleanProperty`.  This is because the status of the `BooleanProperty` was already "Invalid", so there's no change, and no triggering of that `InvalidationListener`.

A few more clicks, and now the `IntegerProperty` is 11, and the `BooleanProperty`, if we called its `getValue()` would report a new value of `true`.  

But nothing happens.

This is because the status never was reset to "Valid" by calling `getValue()`.  

Now, let's change one line of code, the `InvalidationListener` on the `BooleanProperty`:

``` java
overTen.addListener(ob -> System.out.println("Boolean invalid " + counter.getValue() + " " + overTen.getValue()));
```
All we've done here is to add in a call to `getValue()` on the `BooleanProperty`.  So now every time it's invalidated, we're calling `getValue()` and resetting it to "Valid".  

The output now looks like this:

```
Button Clicked
Boolean invalid 8 false
Counter Invalid: 8
Button Clicked
Boolean invalid 9 false
Counter Invalid: 9
Button Clicked
Boolean invalid 10 false
Counter Invalid: 10
Button Clicked
Boolean invalid 11 true
Counter Invalid: 11
```

Now we see that the `BooleanProperty` in invalidated every time the `IntegerProperty` is invalidated, and the `InvalidationListener` on the `BooleanProperty` is triggered.  

One last change, to show another aspect you need to be aware of.  This time we're going to remove the `getValue()` calls on the `IntegerProperty`.  So it will never directly be reset to "Valid":

``` java
overTen.addListener(ob -> System.out.println("Boolean invalid "  + overTen.getValue()));
counter.addListener(ob -> System.out.println("Counter Invalid: "));
```
And this is what the output looks like:
```
Button Clicked
Boolean invalid false
Counter Invalid:
Button Clicked
Boolean invalid false
Counter Invalid:
Button Clicked
Boolean invalid false
Counter Invalid:
Button Clicked
Boolean invalid true
Counter Invalid:
```
You can see that the `InvalidationListener` on the `IntegerProperty` is invoked every time the `Button` is clicked.  This is because, in order to recalculate the value of the `BooleanProperty`, the `Binding` has to implicitly invoke the `getValue()` on the `IntegerProperty`.  So it actually does get reset each time.

It is possible to write a `Binding` that uses an `ObservableValue` as a trigger, but doesn't call `getValue()` on that `ObservableValue` when it's own `getValue()` is called.  This is something you need to watch out for.

## Using ChangeListener

We've seen how "Invalidation" is different from "Changed", and how `InvalidationListeners` work.  But what about `ChangeListener`?

`ChangeListeners` are invoked whenever the value in the `ObservableValue` actually changes.  This means that, behind the scenes, the `getValue()` method of the `ObservableValue` is going to be called.  This is probably why `getValue()` is defined in ObservableValue - which introduces ChangleListener - and not in Observable: You need `getValue()` in order to implement `ChangeListener`.

### An Example

Let's go back to that same example, but modify it to use ChangeListeners instead:

``` java
public class ChangeListenerDemo extends Application {

    private final BooleanProperty overTen = new SimpleBooleanProperty(false);
    private final IntegerProperty counter = new SimpleIntegerProperty(7);

    public static void main(String[] args) {
        launch();
    }

    @Override
    public void start(Stage stage) throws IOException {
        overTen.bind(counter.greaterThan(10));
        overTen.addListener((ob, oldValue, newValue) -> System.out.println("Boolean changed: " + newValue));
        counter.addListener((ob, oldValue, newValue) -> System.out.println("Counter changed: " + newValue));
        Scene scene = new Scene(createContent(), 320, 240);
        stage.setScene(scene);
        stage.show();
    }

    private Region createContent() {
        Button button = new Button("Click Me!");
        button.setOnAction(evt -> {
            System.out.println("Button Clicked");
            counter.set(counter.getValue() + 1);
        });
        VBox vBox = new VBox(button);
        vBox.setPadding(new Insets(30));
        return vBox;
    }
}
```

Just two lines have been changed.  You can see that `ChangeListener` gets both the old value and the new value of the `Property` as parameters.  We're just going to look at the new value here.

The output looks like this:

```
Button Clicked
Counter changed: 8
Button Clicked
Counter changed: 9
Button Clicked
Counter changed: 10
Button Clicked
Boolean changed: true
Counter changed: 11
```
This is quite different.  The `ChangeListener` for the `IntegerProperty` is called every time, but the `ChangeListener` for the `Boolean` is only called once, when the value changes from `false` to `true`.

## How Do They Interact?

Let's see.  Back to the same example, but we'll put both an `InvalidationListener` and a `ChangeListener` on each `Property`.  The `InvalidationListeners` are just going to output that they were triggered, without calling `getValue()`.

``` java
public class ChangeListenerDemo extends Application {

    private final BooleanProperty overTen = new SimpleBooleanProperty(false);
    private final IntegerProperty counter = new SimpleIntegerProperty(7);

    public static void main(String[] args) {
        launch();
    }

    @Override
    public void start(Stage stage) throws IOException {
        overTen.bind(counter.greaterThan(10));
        overTen.addListener((ob, oldValue, newValue) -> System.out.println("Boolean changed: " + newValue));
        counter.addListener((ob, oldValue, newValue) -> System.out.println("Counter changed: " + newValue));
        overTen.addListener(ob -> System.out.println("Boolean invalid "));
        counter.addListener(ob -> System.out.println("Counter invalid "));
        Scene scene = new Scene(createContent(), 320, 240);
        stage.setScene(scene);
        stage.show();
    }

    private Region createContent() {
        Button button = new Button("Click Me!");
        button.setOnAction(evt -> {
            System.out.println("Button Clicked");
            counter.set(counter.getValue() + 1);
        });
        VBox vBox = new VBox(button);
        vBox.setPadding(new Insets(30));
        return vBox;
    }
}
```
The output looks like this:

```
Button Clicked
Boolean invalid
Counter invalid
Counter changed: 8
Button Clicked
Boolean invalid
Counter invalid
Counter changed: 9
Button Clicked
Boolean invalid
Counter invalid
Counter changed: 10
Button Clicked
Boolean invalid
Boolean changed: true
Counter invalid
Counter changed: 11
```
Okay, so both of the `InvalidationListeners` are invoked every time the value in the `IntegerProperty` changes.  This means that both `Properties` are reset to "Valid" by their `ChangeListener` even if the `ChangeListener` isn't triggered on that `Property`.


# Yes, But Which Should You Use?

The truth is, for virtually every use case where you call the `ObservableValue`'s `getValue()` method, there is no practical difference between `ChangeListener` and `InvalidationListener`.  So use whichever you feel more comfortable with.

## I Prefer InvalidationListener

Personally, I find that `InvalidationListener` is a little bit simpler to implement, because there's only one parameter and calling `ob.getValue()` is no more difficult than accessing the `newVal` parameter of `ChangeListener`.

I find that `InvalidationListener` requires you to think a little bit more about the underlying mechanism that triggers the listener, and this is a good thing.  You should understand what's really going on and base your design on that understanding.  Finally, you should be aware the `Bindings` are based on `InvalidationListeners` under-the-hood, and if your `Bindings` are doing strange things, being comfortable with the concepts behind `InvalidationListeners` is going to make it easier to fix the problems.

## Sometimes the Invalidation is More Important Than the Change

There are occasions where having a `Property` become "Invalid" may be more important than seeing the value change.  Perhaps there are elements of you application that aren't represented by `Properties` that need to be re-evaluated at certain times.  In that case, an `InvalidationListener` might be more appropriate than a `ChangeListener`.

And remember, it's very possible that an `InvalidationListener` might trigger even when a `ChangeListener` wouldn't.  

## Use ChangeListener If You Care About the Old Value

Since `ChangeListener` gives you both the old and the new value of the property, if your logic requires that you do something different based on different values the Property might changed from, then `ChangeListener` is the way to go.  Another way of saying this is: **Use ChangeListener if you care about the nature of the change.**
