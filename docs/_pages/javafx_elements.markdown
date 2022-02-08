---
layout: single
title: The Elements of JavaFX
toc: true
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
permalink: /javafx/elements/
skip_link: true
---
# The Elements of JavaFX You Need to Master

## The Nodes
There are large amount of classes in JavaFX arranged in a hierarchy starting with a top level class called, `Node`.  There are a number of layout classes that all work differently, such as `BorderPane`, `ScrollPane`, `HBox`, `VBox`, `GridPane` and `StackPane` which can all be embedded inside of each other to create exactly the layout you want.

There are classes to display images, text or text and images together.  `ListView`, `TreeView` and `TableView` allow collections of data to be displayed on screen (and edited) in highly customizable ways.

For user input, you can choose from `TextField`, `ComboBox`, `ChoiceBox`, `Spinner`, `Checkbox`, `RadioButton`, `Buttons` and `ToggleButtons`.

All of these Nodes have a large number of methods and properties for controlling their presentation and how they act.  Learning how to use them and configure them can be a daunting task.  Luckily, most of the tutorials on the web seem to focus on this aspect of JavaFX, so help is easy to find.

[Read More](/javafx/elements/nodes){: .btn .btn--info}

## Bindings and Observable Objects

This is the Reactive part of JavaFX, and it's critical to understand how this works.  

### Observable Objects

Just like the Nodes, there are a large number of classes and interfaces that extend from the `Observable` interface.  This includes the `ObservableList` and `Property` interfaces which used extensively in just about any JavaFX application.  

`Observables` are wrappers around various data types, and the values inside them can be imperatively accessed through getters and setters.

`Observables` are used in two ways.  The most important is `Bindings`, the other is `ChangeListeners`.

### Bindings

The interface `Property` contains only methods related to binding, and this is the key characteristic of any `Observable` class.  Binding is how a `Property` is connected to one or more other properties.  So that a change in one of those `Properties` will trigger a change in the bound `Property`.  `Binding` is also an interface - which extends `Observable`, and `ObservableValue` - so you can connect any number of `Properties` together into a single value which is contained inside of an `Observable` wrapper, which means it can be used just like a read-only `Property`.

All of the JavaFX `Node` descendants have a variety of properties that can be bound from (or to) properties in other `Nodes`, or custom properties created by the programmer.  This is the core of how a reactive application is built - `Properties` in the GUI are bound to `Properties` in the Data Model, and then application controls the GUI largely by manipulating the state of the Data Model.

### Listeners

`ChangeListeners` are way to manually monitor a `Property` for changes in its value, and then to trigger some code when a change happens.  Generally speaking, `Bindings` are much preferred over listeners, but there are times when only a listener will work.  The reason for this is that **Bindings link "State", while ChangeListeners transform changes in "State" to actions**.  Programming to State is usually better, but events work better in some circumstances, such as when data from external API's need to be loaded on a State change.

[Read More About Listeners](/javafx/listeners){: .btn .btn--info}

## Events and Actions

At its heart, JavaFX is "Event Driven Programming", and even bindings probably boil down to event handling deep inside the JavaFX library.  `Events` can be triggered by `Property` changes, through mouse move movements and clicks, by keyboard actions, and the actions on `Nodes` like `Buttons` and `MenuItems`.  `Events` are captured in the code through "Handlers", and each `EventHandler` specifies code that should be run when the event occurs.

It's important to understand where actions and `Events` are more appropriate than State changes and `Bindings`.  Something like a `Button` click is unmistakably an "Action", and should probably be treated as such.  It would be possible to configure a click handler on a `Button` to toggle a `Property` in State, which is then captured with a `ChangeListener` in an application logic class, but that's probably the most complicated way to program the process.  

## CSS

All of the JavaFX `Nodes` can be styled with cascading style sheets.  These are largely compatible with web standards for CSS, but there are some differences to learn about.  How you implement CSS styling largely becomes a matter of taste.  You can override the standard styling for entire classes of `Nodes`, or you can create custom selectors and attach them to specific `Nodes` in your code.  Or you can do both.

Pseudo-Classes are the best way to represent changes in the State of your application the styling of your GUI.  

[Read About Pseudo-Classes Here](/javafx/pseudo_classes){: .btn .btn--info}

## Handling the GUI Thread

Like virtually every GUI library available, it's really, really bad to do long or blocking operations on the same thread that is maintaining the GUI.  This handled by launching a background thread to do the otherwise blocking operation, and then triggering an event at its completion which will update the GUI.  The GUI thread in JavaFX is called the "FX Application Thread", or "FXAT"

{% include notice type="warning" content="This is probably the number one issue that beginners in JavaFX struggle with." %}

The biggest implication of this is that you cannot program linearly.  Try as hard as you like, but there's no way that you can run anything that looks like a "Function" (ie. input some data and wait for an answer) triggered from the FXAT that uses a background thread.  When your application behaves weirdly and seems laggy or hangs, come back to this paragraph and read it carefully because this is what you're probably trying to do.

JavaFX provides a handy class called "Task", which supplies a runnable which you can pass to a Thread.  When the code in the Task has been completed, it triggers a "Success" Event.  You can supply an Event Handler which be invoked when the "Success" event is triggered.  Just like any Event Handler, it will be run on the FXAT and can therefore update the GUI.  
