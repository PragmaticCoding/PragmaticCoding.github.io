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



## Creating a Project

# Getting Started

# Applications, Stages and Scenes



# The Nodes

## Container Classes

## Input Classes



# Properties and Observables
