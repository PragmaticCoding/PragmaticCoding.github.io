---
title:  "EventHandlers, Listeners and Bindings - What to Use Where"
date:   2024-03-07 00:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/events_and_listeners
Diagram: /assets/posts/MVCI.png
Dispatch1: /assets/elements/EventDispatchChain1.png
Dispatch2: /assets/elements/EventDispatchChain2.png
DemoRunning: /assets/posts/MvciDemoRunning.png
DemoDone: /assets/posts/MvciDemoDone.png
excerpt: Taking a look at how EventHandlers, Listeners, Subscriptions and Bindings are different, and how they should be used in a JavaFX application.
---

# Introduction

One issue that seems to come up quite often is how to connect pieces of your application through JavaFX `Observable` classes.

# State vs Events

Being very, very, very general - and not even directly related to JavaFX - there are two approaches to connecting changes between applications or parts of an application.  One is to track things that happen, Events, and the other is to connect through State.  

Also being very, very general - and still not directly related to JavaFX - it's usually safer to connect via State.

Why?

Because Events are, by their nature, ephemeral.  They happen and they are gone, and if you don't capture them when they happen you're just out of luck.  All Events need some sort of communication structure, or bus, to travel on, and every system that needs to track Events needs to be connected to that bus at the time that the Events happen or they won't be aware of the Events.

On the other hand, State can be checked at any time and it will always give the correct answer.  This means that a system (or subsystem) can query another system about State at the time that it needs the data an get the current answer.  This is independent of whether changes to that State had triggered an Event, or whether that Event is still sitting on a bus waiting to be captured, or even if there are many Events sitting on the bus related to this data.  

For JavaFX programmers, the good news is that JavaFX provides an amazing "State Machine" with all the features that you need to connect parts of your application via State.

## Let's Get One "Issue" Out of the Way

Yes, JavaFX is essentially "Event Driven", and "State" is just an illusion in JavaFX.  

Deep, deep "under-the-hood", JavaFX is using Event methodology to provide the actions that enable a State Machine.  Look down far enough and you'll find `InvalidationListeners` running everything State related.

But, it's probably safe to assume that the core JavaFX libraries are robust, well designed and thoroughly tested and can be treated as essentially foolproof.  We can ignore the underlying Event Driven nature of JavaFX and treat its State features as State - and think of them that way.

# "State" in JavaFX = Bindings

When we think about connecting things via "State" in JavaFX, that almost exclusively means employing `Bindings` as the mechanism.  There are some edge cases that the standard binding tools cannot handle, and in those cases it may be necessary to use a `Listener` or a `Subscription`.  But, for the most part, we're talking about `Bindings`.

When you bind a `Property` to another `ObservableValue`, JavaFX will initialize the bound property to the same value as that of the `ObservableProperty` it's been bound to, and from that point on it will remain synchronized.  No matter what.  You can count on this.

Properties are all over the JavaFX `Node` classes.  `Labels`, `Texts` and `TextFields` all have `TextProperty` which holds the value that's displayed on the screen.  Many of the other input controls have a `ValueProperty` which serves a similar function.  `Regions` have `HeightProperty`, `WidthProperty`, `MinHeightProperty`, `MaxWidthProperty` and `PaddingProperty` amongst many others.  These all control how these `Nodes` are displayed.  

And it goes on.  Any subclass of `Pane` has `Children`, which is an `ObservableList` of all of the `Nodes` directly contained inside of it.  `TableView` and `ListView` have `Items`, which is an `ObservableList` of the data displayed in them.

All of these `Properties` and `ObservableLists` that aren't read-only can be bound to other `Properties` in your Presentation Model or in other `Nodes`.  The read-only `Properties` can be bound from by `Properties` in your Presentation Model or other `Nodes`.  

## All of Your JavaFX Data Should be Encapsulated in Observables

In order to leverage all of these `Properties` in the JavaFX `Nodes`, you need to wrap all of the data in your Presentation Model inside `Properties`, `ObservableValues` and `ObservableLists`.  When you've done this, you can connect these elements of your Presentation Model to the GUI through bindings.  Now your Presentation Model truly becomes a data representation of the "State" of your GUI.  You can read it from outside the GUI, and you can alter it from outside the GUI to change the GUI - without knowing about the GUI implementation.

## Bindings Should be Your "Go To" Solution




# Listeners Convert State Changes to Actions

There are times when it is necessary to perform an action in response to a change in State.  This is what Listeners are primarily intended to do.  There are some edge cases where Bindings cannot do what you want to do, and you may need to use a `Listener` to manually update a value based on an `ObsevableValue`.  These edge cases usually seem to involve `ObservableLists`.  

At times it may not be clear whether the situation calls for a `Listener` or an `Event`.  Imagine that you have a View that has a `Button` in it.  That `Button` needs to trigger some activity all the way down in the business logic - which certainly isn't going to be in the View itself.  You might be able to trigger that activity by updating some value in the Model, and then put a `Listener` on that Model element in the Controller or the Interactor.  But it's also possible for the Controller to supply the View with a `Consumer` that it can invoke to explicitly trigger the activity in the business logic.

My personal inclination is to go with the second approach, and invoke a `Consumer` to trigger the activity.  There are two reasons for this:  First, the `Button` is going to trigger an action through an `EventHandler` in any case.  Converting that action into a state change in order to detect the state change and trigger another action seem, to me, to be the long way around the square.  Secondly, that state change could possibly happen from any number of places other than the `Button` click, and how is the business logic to know that the action is required?  

Another use of `Listeners` is to block changes.  Simply create a `ChangeListener` and in it's code have it check conditions to see if the change is allowed.  If it isn't, then change it back to the old value.  The big thing here is that *might* need to add a flag that says, "I'm putting it back", so that your `ChangeListener` won't detect this change back as a change it needs to block, creating an infinite loop.

## Listeners Don't Launch Event Handlers

An important consideration when using `Listeners` is to understand how JavaFX executes them.  Consider the following code:

``` kotlin
val test = SimpleStringProperty("abc")
test.addListener { observable, oldValue, newValue ->
   println("Old value was: $oldValue, New value is:  $newValue")
}
println("Before the change")
test.value = "XYZ"
println("After the change")
```
Which will generate the following output:
```
Before the change
Old value was: abc, New value is: XYZ
After the change
```
If the code inside the `ChangeListener` was to be run the same as an `EventHandler`, submitted to the FXAT as a job, then we would expect to see the following output:

```
Before the change
After the change
Old value was: abc, New value is: XYZ
```
What this means is that ChangeListener runs in-line with the code that invokes it.  This holds for Bindings as well.  

99% of the time, this is completely irrelevant to your design.  But once in a while you'll have a case where you actually need the `Listener` to execute after everything else, the same way that the screen updates are done after you're code completes.  In those cases, put the body of your `Listener` code in a `Platform.runLater()` call.

# Subscriptions Simplify Listeners

As of JavaFX 21 we now have a new alternative to `Listeners`: `Subscriptions`.

`Subscriptions` are essentially wrappers around `Listeners` but they have two advantages:

1.  Invoking the `subscribe()` function on an `ObservableValue` or `Observable` will return a reference to the `Subscription` which can be used to unsubscribe.
1.  The `subscribe()` functions are easier to use as they require less boilerplate code and fewer passed parameters.

If you are using JFX or later, you should probably use `Subscriptions` instead of `Listeners` 95%+ of the time.  There is virtually no advantage in using a `Listener` over a `Subscription`, and `Subscriptions` will give you simpler, cleaner code.  

# EventHandlers Describe Actions Triggered by Events

`Events` are the "messages" that are passed around your JavaFX application.  At any given time there could be hundreds or even thousands of `Events` active in your JavaFX application.

It's easy to get the impression that an `EventHandler` is "triggered" by the JavaFX object where it is defined, but this isn't technically correct.  That JavaFX object will "fire" an `Event` and that `Event` is passed around through the JavaFX components of the application according to some specific rules.  In most cases, it will land back at the JavaFX object that fired the `Event` (also known as the "source") and if it has an `EventHandler` of the correct type defined on it, that `EventHandler` will be invoked.

This may sound like I'm splitting hairs, but this can be an important concept to understand.  JavaFX objects will often be passed `Events` that they did not trigger.  JavaFX objects can, therefore, act on `Events` that they did not trigger.  JavaFX objects can also intercept `Events` that they did not trigger and prevent them from reaching the JavaFX object that fired the `Event`.
