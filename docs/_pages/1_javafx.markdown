---
layout: single
title: JavaFX
toc: true
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
permalink: /javafx/javaFX/
rube: /assets/images/rube_goldberg.png
---

## What's JavaFX?

JavaFX is the modern API for creating interactive desktop applications in Java.  While it doesn't
replace Swing, it represents a significant upgrade for the programmer as it uses reactive programming.

## Is it Hard to Learn?

It can be challenging.  JavaFX has never been very popular, as desktop applications have largely gone out of style, so there simply isn't a great wealth of resources out there to learn how to build applications using JavaFX.  However, if you've had experience using frameworks like React or Jetpack Compose, which are also based on reactive principles, it should be easier for you to learn.

### Don't use ScreenBuilder or FXML

 Okay, so this is controversial and a lots of people disagree with me.  I don't use ScreenBuilder and FXML and I strongly recommend that beginners avoid it.

 What are these things?

 "ScreenBuilder" is the graphical, drag-and-drop, tool for creating screen layouts.  The output from ScreenBuilder is a text file in an XML-like format called "FXML".  JavaFX contains objects and methods to load FXML files into your application and merge them with Java code.

 It is much better to learn how to build your screen with code.  It's actually quite easy, and if you apply DRY (Don't Repeat Yourself), you'll quickly build up a library of builder and helper methods, as well as custom classes, that will strip about 90% of the boilerplate code out of your layout logic.  And this is even easier if you're using Kotlin!  The resulting pure Java/Kotlin code is easier to understand and maintain than any FXML ever will be.

## Reactive programming

From [Wikipedia](https://en.wikipedia.org/wiki/Reactive_programming):

> Reactive programming is a declarative programming paradigm concerned with data streams and the propagation of change....

> For example, in an imperative programming setting, a := b + c would mean that a is being assigned the result of b + c in the instant the expression is evaluated, and later, the values of b and c can be changed with no effect on the value of a. On the other hand, in reactive programming, the value of a is automatically updated whenever the values of b or c change, without the program having to explicit re-execute the statement a := b + c to determine the presently assigned value of a.

**This exactly describes how JavaFX is implemented, and how you should approach building JavaFX applications.**  Somehow, this seems to have been completely missed by the programming world in general, and you won't find JavaFX included in the Wikipedia list of reactive frameworks, and no mention of the reactive nature of JavaFX is included in the [Wikipedia page about JavaFX](https://en.wikipedia.org/wiki/JavaFX).

![A Rube Goldberg Machine]({{page.rube}}){:class="img-responsive" style="float: right" width="440" padding="20px"}
Reactive programming seems to be a challenge for many programmers.  

Much of your GUI code is layout and configuration, and sprinkled in with that are code snippets that define the reactive elements.  The layout code is executed immediately, but the reactive snippets are only triggered when the reactive data streams are modified.


The resulting program feels a bit like a [Rube Goldberg machine](https://en.wikipedia.org/wiki/Rube_Goldberg_machine) in code, or like setting up dominoes to fall over.


## Learn More

If you want to learn how build desktop applications with JavaFX, this website has a lot of information that can help you get started.  Check back often, as I plan to add a lot more content on this subject.

#### The Elements of JavaFX

First off, take a look at [The Elements of JavaFX](/javafx/elements).  This will give you an overview of all the things you'll need to be familiar with in order to write good JavaFX applications.  You don't need to master them all at once before you start, but it's a good idea to be at least aware of how all of theses things fit together.

#### Application Structure

Where do you start?  What should a JavaFX application look like?

Check out my blog post: [JavaFX Without FXML](/javafx/nofxml).  This will give you step by step instructions for building a complete (but simple) application with data input, showing you some good practices to follow and how to keep everything clean and easy to read.

Next, [Using MVC with JavaFX](/javafx/MVC_In_JavaFX), will show you how to add some structure to your application, using Model-View-Controller.  MVC works really well with JavaFX, and is a natural fit to the Reactive nature of JavaFX.
