---
title:  "EventHandlers, Listeners and Bindings - What to Use Where"
date:   2024-04-01 00:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/events_and_listeners
Diagram: /assets/posts/MVCI.png
Dispatch1: /assets/elements/EventDispatchChain1.png
Dispatch2: /assets/elements/EventDispatchChain2.png
DemoRunning: /assets/posts/MvciDemoRunning.png
DemoDone: /assets/posts/MvciDemoDone.png
excerpt: Taking a look at how EventHandlers, Listeners, Subscriptions and Bindings are different, and how they should be used in a JavaFX application.
---

# Introduction

The more that I work with JavaFX, the more that I am convinced that it is one of the best frameworks for building "Reactive User Interfaces" that's out there.  But what is a "Reactive User Interface"?

A reactive user interface is one where there is a data representation of the state of the UI (the "Presentation Model") which is synchronized with the behaviour and appearance of the UI.  This data acts as a pipeline to connect the UI and the business logic without either having any knowledge of the other.  The goal is to reduce coupling between the business logic and the implementation of the UI, while maintaining functionality.

Let's look at how this might work:  

Let's say that in our Presentation Model we have a `Boolean` element called `okToSave`.  First, notice that it's not called `saveButtonEnabled`, or `nameFieldCompleted`.  It needs to have a name which is implementation agnostic from either end of the pipeline.  `saveButtonEnabled` implies the implementation of a "Save" button, while `nameFieldCompleted` implies that it's dependent on some business rule involving some "Name" field having a value.  

The `okToSave` element implies a "contract" in the system that the value in `okToSave` will, at any given time, represent whether or not the UI is in a state where "Save" is a valid operation.  This determination is probably going to be made by the business logic.  If I was inclined to use comments - which I generally avoid - this is one of the places where I would use them, and probably in the form of JavaDocs for the getter/setter functions for this element of the Presentation Model: indicating that the business logic is expected to somehow keep this value current and valid at all times.

In a reactive framework, `okToSave` needs to implement the "Observer Pattern" in some fashion.  In JavaFX, we'd make it a `BooleanProperty`.  At the GUI end of the pipeline, we treat it like an `Observable`, meaning that we never deal directly with its value but we connect it as an `Observable` to some element(s) of the GUI. At the business logic end of the pipeline, we are more concerned with `okToSave` as a discrete data element.  It's possible that we may want to monitor it for changes, in order to trigger some business logic, but it's far more likely that we're going to update its value directly, as part of some business logic.

The most important point is that both ends of the pipeline - the UI and the business logic - have independent views of the same data and deal with it in different ways.  And both ends of the pipeline have no knowledge of how it's used or maintained at the other end.  The UI could simply connect it to the `disabled` property of the "Save" button, while the business logic might set it up by connecting it to the values of several other elements in the Presentation Model using some kind of mapping.  But both ends can do their job without any knowledge what-so-ever about what's going on at the other end.

The opposite technique of "reactive" is "imperative" programming.  In this kind of programming, you pass a piece of information to the UI and say, "Now do this with it", or the UI passes back some information and says, "Take this info and check it for me", or something like that.  You certainly can write applications this way, and in JavaFX, but it's much, much harder to keep your business logic decoupled from the UI without a lot of really fussy code.

Reactive programming feels more like setting up dominoes in elaborate patterns so that they'll interact with each other in a certain way when you knock one over.  In other words, you're telling your layout how to *react* to changes in the state of the application.  

Okay, there's the background and the reason *why* we might want to design our application in a reactive fashion.  Now, let's talk about how to do that.  One issue that seems to come up quite often is how to connect pieces of your application through JavaFX `Observable` classes.  That's the subject of this article.

# State vs Events

Being very, very, very general - and not even directly related to JavaFX - there are two approaches to connecting changes between applications or parts of an application.  One is to track things that happen - events or messages - and the other is to connect through State.  

Also being very, very general - and still not directly related to JavaFX - it's usually safer to connect via State.

Why?

Because Events are, by their nature, ephemeral.  They happen and they are gone, and if you don't capture them when they happen then you're just out of luck.  All events need some sort of communication structure, or bus, to travel on, and every system that needs to track events needs to be connected to that bus at the time that the events happen or they won't be aware of the events.  There are lots of libraries and systems out there designed to improve message delivery and to make it "guaranteed", and there's a reason for this - event/message handling is intrinsically fraught with difficulties.

On the other hand, State can be checked at any time and it will always give the correct answer.  This means that a system (or subsystem) can query another system about State at the time that it needs the data and can get the current answer.  This is independent of whether changes to that State had triggered an event, or whether that event is still sitting on a bus waiting to be captured, or even if there are many events sitting on the bus related to this data.  

What this means is that, generally speaking, you should prefer to wire your system together through state when possible.  There are occasions when events are the way to go, and we'll talk about that later.

For JavaFX programmers, the good news is that JavaFX provides an amazing "State Machine" with all the features that you need to connect parts of your application via State.

## Let's Get One "Issue" Out of the Way

Yes, JavaFX is essentially "Event Driven", and "State" is just an illusion in JavaFX.  

Deep, deep "under-the-hood", JavaFX is using Event methodology to provide the actions that enable a State Machine.  Look down far enough and you'll find `InvalidationListeners` running everything State related.

But, it's probably safe to assume that the core JavaFX libraries are robust, well designed and thoroughly tested and can be treated as essentially foolproof.  We can ignore the underlying Event Driven nature of JavaFX and treat its State features as State - and think of them that way.

# "State" in JavaFX = Bindings

When we think about connecting things via "State" in JavaFX, that almost exclusively means employing `Bindings` as the mechanism.  There are some edge cases that the standard binding tools cannot handle, and in those cases it may be necessary to use a `Listener` or a `Subscription`.  But, for the most part, we're talking about `Bindings`.

When you bind a `Property` to another `ObservableValue`, JavaFX will initialize the bound `Property` to the same value as that of the `ObservableProperty` it's been bound to, and from that point on it will remain synchronized.  No matter what.  You can count on this.

`Properties` are all over the JavaFX `Node` classes.  `Labels`, `Texts` and `TextFields` all have `TextProperty` which holds the value that's displayed on the screen.  Many of the other input controls have a `ValueProperty` which serves a similar function.  `Regions` have `HeightProperty`, `WidthProperty`, `MinHeightProperty`, `MaxWidthProperty` and `PaddingProperty` amongst many others.  These all control how these `Nodes` are displayed.  

And it goes on.  Any subclass of `Pane` has `Children`, which is an `ObservableList` of all of the `Nodes` directly contained inside of it.  `TableView` and `ListView` have `Items`, which is an `ObservableList` of the data displayed in them.

All of these `Properties` and `ObservableLists` that aren't read-only can be bound to other `Properties` in your Presentation Model or in other `Nodes`.  The read-only `Properties` can be bound from by `Properties` in your Presentation Model or other `Nodes`.  

## All of Your JavaFX Data Should be Encapsulated in Observables

In order to leverage all of these `Properties` in the JavaFX `Nodes`, you need to wrap all of the data in your Presentation Model inside `Properties`, `ObservableValues` and `ObservableLists`.  When you've done this, you can connect these elements of your Presentation Model to the GUI through bindings.  Now your Presentation Model truly becomes a data representation of the "State" of your GUI.  You can read it from outside the GUI, and you can alter it from outside the GUI to change the GUI - without knowing about the GUI implementation.

The most effective way to do this is to bundle all of your Presentation Model elements into a single class which is just a POJO with JavaFX `Observable` types as the fields.  Don't put any logic or linkages in there, just the fields.

## Bindings Should be Your "Go To" Solution

When designing reactive JavaFX applications, your first approach to any issues should be, "How can I do this with Bindings?".  This may require that you reorganize your data models or your GUI `Nodes`.  That's fine, it probably will result in a better, more logical data structure anyway.  

You **need** to understand how `Bindings` work and all of the techniques related to `Bindings`.  You can learn a lot of what you need to know from the articles listed on [this](https://www.pragmaticcoding.ca/javafx/elements/observables) page.  

Sometimes, usually related to `ObservableLists`, you will come across a situation where a `Binding` won't work, or you cannot figure out how to write one that will work.  In those instances you may have to use a `Subscription` or a `Listener` of some sort.  


# Listeners Convert State Changes to Actions

There are times when it is necessary to perform an action in response to a change in State.  This is what Listeners are primarily intended to do.  There are some edge cases where `Bindings` cannot do what you want to do, and you may need to use a `Listener` to manually update a value based on an `ObservableValue`.  These edge cases usually seem to involve `ObservableLists`.  

At times it may not be clear whether the situation calls for a `Listener` or an `Event`.  Imagine that you have a View that has a `Button` in it.  That `Button` needs to trigger some activity all the way down in the business logic - which certainly isn't going to be in the View itself.  You might be able to trigger that activity by updating some value in the Model, and then put a `Listener` on that Model element in the Controller or the Interactor.  But it's also possible for the Controller to supply the View with a `Consumer` that it can invoke to explicitly trigger the activity in the business logic.

My personal inclination is to go with the second approach, and invoke a `Consumer` to trigger the activity.  There are two reasons for this:  First, the `Button` is going to trigger an action through an `EventHandler` in any case.  Converting that action into a state change in order to detect the state change and trigger another action seems, to me, to be the long way around the square.  Secondly, that state change could possibly happen from any number of places other than the `Button` click, and how is the business logic to know that the action is required?  

Another use of `Listeners` is to block changes.  Simply create a `ChangeListener` and in its code have it check conditions to see if the change is allowed.  If it isn't, then change it back to the old value.  The big thing here is that you *might* need to add a flag that says, "I'm putting it back", so that your `ChangeListener` won't detect this change back as a change it needs to block, creating an infinite loop.

## Listeners Don't Launch Event Handlers

An important consideration when using `Listeners` is to understand how JavaFX executes them.  The way that you define them looks very, very similar to the way that you attach an `EventHandler` to a `Node` event.  But they execute very differently.

Consider the following code:

``` kotlin
val test = SimpleStringProperty("abc")
test.addListener { observable, oldValue, newValue ->
   println("Old value was: $oldValue, New value is:  $newValue")
}
println("Before the change")
test.value = "XYZ"
println("After the change")
```
Which (perhaps surprisingly) will generate the following output:
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
But it doesn't, and this means that the ChangeListener runs in-line with the code that invokes it.  This holds for Bindings as well.  

99% of the time, this is completely irrelevant to your design.  But once in a while you'll have a case where you actually need the `Listener` to execute after everything else, the same way that the screen updates are done after you're code completes.  In those cases, put the body of your `Listener` code in a `Platform.runLater()` call.

## Subscriptions Simplify Listeners

As of JavaFX 21 we now have a new alternative to `Listeners`: `Subscriptions`.

`Subscriptions` are essentially wrappers around `Listeners` but they have two advantages:

1.  Invoking the `subscribe()` function on an `ObservableValue` or `Observable` will return a reference to the `Subscription` which can be used to unsubscribe.
1.  The `subscribe()` functions are easier to use as they require less boilerplate code and fewer passed parameters.

If you are using JFX or later, you should probably use `Subscriptions` instead of `Listeners` 95%+ of the time.  There is virtually no advantage in using a `Listener` over a `Subscription`, and `Subscriptions` will give you simpler, cleaner code.  

To learn more about `Subscriptions` you can read my article [here](https://www.pragmaticcoding.ca/javafx/subscribe_and_map#observablevaluesubscribe-and-observablesubscribe).

# EventHandlers Describe Actions Triggered by Events

`Events` are the "messages" that are passed around your JavaFX application.  At any given time there could be hundreds or even thousands of `Events` active in your JavaFX application.

It's easy to get the impression that an `EventHandler` is "triggered" by the JavaFX object where it is defined, but this isn't technically correct.  A JavaFX object will "fire" an `Event` and that `Event` is passed around through the JavaFX components of the application according to some specific rules.  In most cases, it will land back at the JavaFX object that fired the `Event` (also known as the "source") and if it has an `EventHandler` of the correct type defined on it, that `EventHandler` will be invoked.

This may sound like I'm splitting hairs, but this can be an important concept to understand.  JavaFX objects will often be passed `Events` that they did not trigger.  JavaFX objects can, therefore, act on `Events` that they did not trigger.  JavaFX objects can also intercept `Events` that they did not trigger and prevent them from reaching the JavaFX object that fired the `Event`.

You'll see a lot of example code that calls `Event.getSource()` to determine the `Node` that called the `EventHandler`.  Most of the time, this will be correct, but depending on the manner in which the `EventHandler` is called, it's possible that the source of the `Event` was some other `Node`.  

One of the main places that you'll see this used is in the "Dispatch `EventHandler`" pattern.  This is where a single `EventHandler` is written, and every `Node` in the layout invokes that same `EventHandler`.  Inside the `EventHandler` there is some code that checks `Event.getSource()` and then branches based on which `Node` generated the `Event`.  

Most of this is a waste of time and code.  With lambda expressions you can write a one line `EventHandler` that calls a specific method that does what you want.  Something like this:

``` kotlin
button.setOnAction{ event -> someMethod() }
```
in Kotlin, we can ditch the `event ->` if we are not using it (or just let it be `it` if we are using it), so the code becomes:

``` kotlin
button.setOnAction{ someMethod() }
```

There's something about FXML that makes the "Dispatch `EventHandler`" attractive to programmers.  It's probably just copypasta from on-line examples that define a single "#handleClick" in the FXML which corresponds to a dispatch method in their FXML Controller.

If you are writing an `EventFilter` then you are probably calling it from the `Node` that is neither the source nor the target of the `Event`.  In that case, you might want to know which `Node` is the source or the target.  This is one of the few situations where you should be checking the source or the target.

Usually `Events` are used when something "happens" in your GUI or your JavaFX environment.  This is most often when the user takes some action, like moving the mouse or clicking a mouse button.  But `Events` can also be fired when the GUI does something like showing the pop-up of a `ComboBox` or when a `Task` completes.  Note, however, that these really do represent an occurrence of something happening - a moment in time.  

Presumably, these things that happen are things that you want to capture and act upon - otherwise why would you create an `EventHandler` for them?  

Finally, you can manually fire `Events`.  You can use this to trigger already defined `EventHandlers` on other `Nodes`.  For instance, you could have a layout with several `Buttons`, each of which has its own function defined in it `ActionEvent` handler.  One of those `Buttons` could be a "do everything" `Button`, and its `EventHandler` fires `ActionEvents` that target the other `Buttons`, causing their `ActionEvent EventHandlers` to execute.  You could also define your own `Event` types, and use the `Event` transport system to pass messages around your layout.

## Task Events

The last kind of `Event` to think about are those that are generated by `Tasks`, especially the `OnSucceeded Event`.  

This `Event` is the standard way to get back on the FXAT after running a job on a background thread.  The important thing to remember is that this `Event` can, like any other `Event` happen at any time.  This means that you cannot assume any temporal relationship between launching a `Task` on a background thread and the `OnSucceeded Event` being fired.  It might happen almost immediately.  It might take seconds, or minutes.  It might never fire.  

And in between; dozens or hundreds or even thousands of other `Events` might have fired in your GUI.  Or not.

What you have to think about, in terms of your GUI, is what "happenings" are not going to be allowed whilst your `Task` is running, and how are you going to stop them?  Most often, when you define and execute a `Task` you're already in "Action Mode"; either by calling an `EventHandler` or code inside a `Listener`.  Whatever code is closest to that trigger is probably the correct place to deal with disabling the GUI elements that you don't want active while the `Task` is running.  This is probably the `EventHandler` or `Listener` code itself.

But on the other hand, `Tasks` are usually created to deal with stuff that needs to run on a background thread because they are likely to block, or might just take a long time to complete.  This is usually because you're invoking something that's outside of your GUI like calling an external API.  

And that stuff doesn't belong anywhere near your layout code.  

Which means that your `Tasks` are probably going to be defined outside of your View, and won't have access to the inside elements of the View.  As a matter of fact, that `OnSucceeded Event` is probably the only `Event` type that you'll use that occurs *outside* of your View.

It will probably need to do two things:  Update your Presentation Model according to business logic, and re-enable the GUI elements that were disabled when the `Task` was launched.  

The first is fairly easy, the second requires some finesse.  Probably the most direct method is to provide a `Callback` of some sort for the `Task` to invoke from the `OnSucceeded EventHandler`.  This `Callback` is defined in the `EventHandler` or `Listener` code that invokes the `Task` creation outside the View, and passed along to it.  

# When To Use What

OK, and this is going to sound obvious and banal, but: **Actions are actions and state changes are state changes**.

You need to keep this in mind all the time.  Things that are "happenings" in your GUI, like mouse clicks or `Node` lifecycle events, pretty much have to result in code defined inside an `EventHandler` executing.  Even if all you are going to do in that code is to change a value in your GUI State.  But things that aren't "happenings" shouldn't generally result in imperative action-style code running.

There are two main considerations:

* Most importantly, if your GUI has put you into "Action Mode", then respond with an action.  Write imperative code that does whatever you need your GUI or business logic to do in a way that you can invoke it directly.  

* On the other hand, state changes are usually best dealt with by setting up `Bindings` so that you can forget all about them and let them work automatically.  If you cannot figure out how to do it with a `Binding`, then use a `Listener` or a `Subscription` so that you can write the imperative code that maintains the connections you want between the data elements.  

If what you're doing falls outside these two situations, then need to have a good reason to do it that way.  Specifically, this means turning an action into a state change, or a state change into an action.  Both are possible, even easy to do.  But should you?  

* Some elements of JavaFX will only respond to actions.  For instance, invoking a `PseudoCLass` state change has to be done via a method call - that's an action, and you can't use `Bindings` to make it happen.  Likewise, programmatically showing the pop-up of a `ComboBox` requires a method call - so that's an action too.

* Sometimes all that an action needs to do is to update State.  Something like a `Button` click that just increments a counter in your Presentation Model - that's defined in the `OnAction EventHandler` of your `Button` and would essentially turn an action into a state change.

Keep in mind that these considerations really apply to your *application* code, meaning code that is specific to your application or instance of a framework.  If you can extrapolate some case of using a state change to trigger an action (or the reverse) into some generic utility code, then you ignore the transition from one type to another.

Consider the example of how a `PseudoClass` requires an action to make the method call to transition the state of a `PseudoClass` on a `Node`.  We can wrap that action up into a custom class extending `BooleanProperty` (well, `BooleanPropertyBase` actually) like this:

``` kotlin
class PseudoClassProperty(private val node: Node, private val pseudoClass: PseudoClass) : BooleanPropertyBase() {
    override fun getBean() = node
    override fun getName(): String = pseudoClass.pseudoClassName

    override fun invalidated() {
        super.invalidated()
        node.pseudoClassStateChanged(pseudoClass, value)
    }
}
```

So now we can simply bind an instance of this `PseudoClassProperty` to whatever `Properties` in our Presentation Model that we want to, and we don't have to worry about the action nature of the state transition handling because it's not part of our application code any more, it's in some utility library.  In our application code, it's just a `Binding`.

# Conclusion

Programmers are used to linear imperative programming.  That means, call a method and get an answer.  Use that answer to do some more work, maybe call some more methods and get some more answers.  Interact with a persistance layer to get and save data.  That kind of thing.

This approach fails badly when it meets a GUI framework.  Mostly because GUI's do not work in a linear imperative manner.  Any number of "things" can happen at any given time, and your GUI needs to react to these "things" in real time.  This means that you generally cannot wait for an answer.  So if you're going to go off to an external API to fetch some data, you cannot write that code like a `Function` where you call it and then you recieve the answer as a return result.  

It's the "linear" part that causes the problem.  Everyone eventually experiences, "I wrote my process with update messages for the GUI but nothing happens.  Then the process ends and all of the messages appear at once at the end!".  That's because you were thinking linearly and wrote your process to return an answer to the calling code, like a `Function`.

Programming with a GUI requires a paradigm shift.  

In JavaFX, that shift should result in you approaching your application as a reactive framework.

Writing your application becomes first about defining the relationships between the data and the `Nodes` in the GUI.  Or between the data in the `Nodes` and the data in other `Nodes`.  Secondly, it becomes about defining how the application will react when certain things "happen" in the application.  Usually, those "happenings" are `Events`, but sometimes they are changes in the data.

Basically, you are defining how your application will react to changes and events that can happen, literally, at any time and in any order.  You cannot get around this, because this is the way that JavaFX works.  So embrace the reactive tools that JavaFX gives you and think in a reactive way.
