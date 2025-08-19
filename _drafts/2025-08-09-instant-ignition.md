---
title:  "Reactive JavaFX: Instant Ignition"
date:   2025-08-09 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/instant-ignition
ScreenSnap0: /assets/elements/ConstrainedProperty0.png
ScreenSnap1: /assets/elements/ConstrainedProperty1.png
ScreenSnap2: /assets/elements/ConstrainedProperty2.png
ScreenSnap3: /assets/elements/ConstrainedProperty3.png
ScreenSnap4: /assets/elements/ConstrainedProperty4.png
ScreenSnap5: /assets/elements/ConstrainedProperty5.png
ScreenSnap6: /assets/elements/ConstrainedProperty6.png
ScreenSnap7: /assets/elements/ConstrainedProperty7.png
ScreenSnap8: /assets/elements/ConstrainedProperty8.png
ScreenSnap9: /assets/elements/ConstrainedProperty9.png
ScreenSnap10: /assets/elements/ConstrainedProperty10.png

Diagram: /assets/elements/ListProperties.png
OLArticle: /javafx/elements/observable-classes-lists
ABGuide: /beginners/intro
GitHubLink: https://github.com/thomasnield/ConstrainedProperty

JavaDocOL: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/ObservableList.html
JavaDocFXC: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/FXCollections.html

excerpt: "Are you new to JavaFX and you want to get started on the right foot with minimal fuss?  Do you prefer to learn as you go along but don't know where to start?  Then this article is for you."
---

# Introduction

Maybe my [Absolute Beginners Guide to JavaFX]({{page.ABGuide}}) course isn't for you.  That's fine.  Maybe you prefer to just jump in with both feet and get programming your own stuff as soon as possible.  That's how I feel with most things, too.

But...

You should know that the JavaFX library is huge, comprehensive and extensible.  There's stuff in there that will take you years to find for yourself.  

The biggest impediment that all beginners to JavaFX have is that they just don't know what they don't know.  


## Reactive JavaFX

Let's talk about "Reactive JavaFX".  What is it?

From Wikipedia:

> In computing, reactive programming is a declarative programming paradigm concerned with data streams and the propagation of change.

Which is not too helpful by itself.  In the context of GUI design, this translates to having a mutable data representation of the state of the application which the GUI synchronizes with.  Changes to this data are reflected automatically in the GUI, and changes to the GUI are automatically reflected in the data representation of State.  

In a practical sense, for JavaFX, this means that you create a static layout that acts dynamically to respond to changes in the Presentation Data.  This is done by creating your Presentation Model out of `Observable` data classes, and then linking them to elements of your layout.

When you see the scope of the `Observable` library in JavaFX, and when you realize that virtually all of the data elements of every screen `Node` are stored internally as `Observable` classes themselves, then you'll understand that JavaFX was designed - from the ground up - to be used to build Reactive GUI's.

This guide is going to focus on how to use JavaFX as a Reactive GUI framework.  It's easier and better than approaching GUI design in an imperative fashion, and results in simpler design.

## Beginning Concepts

Before we get looking at the nuts and bolts, there are a few basic ideas that you'll need to understand.

## Creating Layouts

A lot of what we're going to look at centres around creating layouts.

### Don't Use FXML

This used to be an outlier opinion of mine, but I'm seeing more and more programmers stating that they don't bother with FXML any more.  

I'm going to be more emphatic here, though.  If you're just starting out with JavaFX, and you want to understand as much as you can as quickly as you can, then FXML is going to get in your way.  There's a level of complexity to it that is a huge stumbling block to most beginners.  You can avoid that by simply not using FXML.

You're not going to be able to use SceneBuilder to do a "drag and drop" design, but, trust me, you won't miss it.

Can you design a Reactive GUI with FXML.  Absolutely, but it's way more difficult and any advantage that you might gain from SceneBuilder's "drag and drop" you'll give back right away doing the supporting programming.

### Use Builder Methods

Understand that there is "layout" and there is "configuration".  While these are linked, they need to be kept apart as much as possible.

Try to keep your layouts as purely "layout" as possible.  When a programmer looks at your layout code, they should be able to picture the layout, and not get bogged down mentally stripping away the code that doesn't contribute to that.  

### Don't Repeat Yourself



## Creating a Project

# Getting Started

# Applications, Stages and Scenes

When you start out with your first JavaFX application you need to deal with three classes right away...

## The Application Class

All JavaFX applications have (essentially) a Main class that extends `Application`.  The `main()` function needs to call `Application.launch()`, which will start up the JavaFX infrastructure.  Eventually, a method called `Application.start()` will be called.

`Application.start()` is the method where you customize how your application will initialize its GUI, and this is typically the only method in `Application` that you will need to customize.  `Application.start()` is passed one parameter, a `Stage`.  So we'll look at that first.

## The Stage Class

For all practical purposes, a `Stage` corresponds to a "Window" on your screen.  You can customize how it looks, to an extent, and you can add a method to run when it's shown, hidden, or closed.  

Most importantly, a `Stage` is designed to hold a `Scene` which is how you define its contents.

## The Scene Class

A `Scene` is a wrapper class that holds your layout.  It has some `Properties` and methods that it shares with the layout classes, but for the most part, you'll just use `Scene` as an intermediate between the `Stage` and the layout.  The `Stage` holds the `Scene` and the `Scene` holds the layout.  

The most common operations that you'll likely do on a `Scene` are to set it's width and height through its constructor, and to populate its content - either through a constructor parameter, or by calling `Scene.setRoot()`.

## Application.start()

The best way to view all this is as administrative boilerplate that you just have to get through to get running.  My preference is to put as little as possible in `Application.start()`.  This going to boil down to configuring the Stage, creating a Scene, call an external method to create the layout put the result in the Scene, and extecute `Stage.showAndWait()`.

It should look something like this:

``` java
public class HelloWorld extends Application {

    @Override public void start(Stage stage) {
        Scene scene = new Scene(createContent());

        stage.setTitle("Welcome to JavaFX!");
        stage.setScene(scene);
        stage.sizeToScene();
        stage.show();
    }

    public static void main(String[] args) {
        Application.launch(args);
    }
}
```
This is pulled from the JavaDocs for `Stage`, and I've just removed the code that created the layout right in `Application.start()`.  It now calls `createContent()`, which isn't defined in this example.

If you have much more than this in your `Application` class, then you're doing it wrong.

# The Nodes

JavaFX has a complicated hierarchy of classes that can be placed into layouts.  At the very top of this hierarchy is the abstract class called `Node`.  As a short hand, we'll just call anything that you can put into a layout a `Node` instead of the tedious "`Node` subclass".  While `Node` has a few `Properties` and methods that are used or implemented in its subclasses, it's not very interesting in itself.  So there's no point in spending any time on it.

Roughly speaking, all of the rest of the classes can be classified as either "Containers" or "Controls".  So we'll look at them this way.

## Container Classes

Most of the container classes are subclasses of `Pane`, including `Pane` itself.  All of these classes have a method called `getChildren()` which will return an `ObservableList` of `Nodes` that are *directly* inside the container.  You can modify this list, and this is how you add new `Nodes` to a container.  It will generally look like this:

``` java
  Pane somePane = new Pane();
  pane.getChildren().add(new Label("Hello World"));
```
There are some very common container classes that you will use all the time.  Each of these differs in how it arranges its child `Nodes`.  They also have some other characteristics that make them unique.

The key to creating layouts is to use the correct container classes to get the look that you want.  You can put container classes inside other container classes to achieve complicated layouts.

Let's look at the different types of containers available, in a rough order of usefulness:

### HBox and VBox

These two classes are very similar in how they work.  Each arranges the child `Nodes` in a strip, with `HBox` having the strip go to the right, and `VBox` having it go straight down.  

One of the nicest things about these two containers is that the respect the boundaries of their child `Nodes`.  So if a child `Node` grows, then all of the other child `Nodes` will be shoved over to the right or down to make space.  This means you never have to worry about children overlapping each other.

### StackPane

`StackPane` arranges its children on top of each other (coming out of the screen) and centred.  This is useful if you just have one `Node` and you want it centred in an area of the screen, or if you want to create a composite `Node` of some sort.  It's especially useful if you have a number of `Nodes` that you want to share the same space, but only have one visible at a time.  

### BorderPane

`BorderPane` is a bit unusual in that it does support `getChildren()` but you'll never call it.  Instead, it has 5 regions: top, bottom, left, right and center that you put `Nodes` into via `setTop()`, `setBottom()` and so on.  

This is a very useful layout class for you entire application, or for a window.  You can put header information in the top, and status, buttons or messages in the bottom, and then your main content into the center.  

Each area can only hold one direct child `Node`.  So if you want to put more than one thing into the top area, for intstance, you'll need to put those `Nodes` into some other container - even another `BorderPane`.

### Pane

`Pane` is a bit like `StackPane` in that it piles the child `Nodes` on top of each other coming out of the screen, but `Pane` stuffs them all into the top left corner.

One thing you don't ever want to do is use `Pane` and then use `Node.translateX()` or `Node.translateY()` to move the children into place.

### TilePane and FlowPane

These two classes work like `HBox` or `VBox` but wrap into new columns or rows when they've used up all of their vertical or horizontal space.  `TilePane` differs from `FlowPane` in that it sets all of the child `Nodes` to be the same size.  This results in all of the children arranging themselves into neat columns and rows.

### GridPane

Now we are into the realm of overused, but less useful container classes.

`GridPane` allows you to define a number of columns and rows and put individual `Nodes` into "cells" defined by row and column.  This is useful if you have a strict need for **both** rows and columns to be aligned in some manner.  This does happen, but perhaps not as often as you might think.  

In reality, `GridPane` is almost always a giant pain to deal with.  Both row and column have to be explicitly declared for each `Node` and if you want to insert a new row into your `GridPane` then you'll have to move every `Node` in every row below it down by one to make space.  In most cases, you can get a result that looks as good as or better than `GridPane` by nesting `HBoxes` inside a `VBox` or visa versa.  

### AnchorPane

`AnchorPane` works just like `Pane` except you can "anchor" the child `Nodes` to one or more sides of the `AnchorPane`.  This is useful when you need it, but you need it only rarely.

Lots and lots of beginners treat `AnchorPane` just like `Pane`.  They also tend to make `AnchorPane` the root `Node` of all of their layouts - perhaps thinking that "anchor" means the "anchor for the layout".  It could be that it's the first container class alphabetically, and SceneBuilder always defaults to it?  In any event, they'll have an `AnchorPane` containing nothing but a `BorderPane` without anchoring it to any sides.

## Control Classes

There is a class called `Control` that virtually all of the remaining `Nodes` extend.  Curiously `Control` itself extends from `Region` which is also the parent of `Pane`.  This means that most of the `Control` subclasses are actually containers, although we seldom think of them that way.  

### Label


### Button


### RadioButton


### CheckBox


### TextField

### ComboBox and ChoiceBox



## Lists and Tables




# Properties and Observables

## Properties

## ObservableLists 

## Bindings

## Listeners
