---
title: The Absolute Beginners Guide to Reactive JavaFX
short_title: Introduction
permalink: /beginners/intro
excerpt: Introducing a course for programmers starting out to learn JavaFX
---

# A Short Introduction

This is a guide to help you learn how to build JavaFX applications the right way.  It's not a reference guide, but an instructional guide intended to be read in order.  Depending on your learning style, it's probably a good idea to follow along with the guide in your IDE, trying out the examples and experimenting with some of the concepts along the way.

I've tried to keep each part short and easily digestible, so that you can stop at the end of any part and pick later from the next part.

My hope is that you fully understand the content of each part before moving on to the next.  If you are confused about anything you can usually research in on the web, or find it in the JavaDocs for the components involved.  If you are really stuck, you can send me an email and I'll help.  In any event, each part of this guide is written with the assumption that the content that preceded it has been fully understood.

# What We'll Build

To start with, we'll build a "throw-away" application to learn some basic concepts.  This will start out as a simple "Hello World" application that we'll add some styling and user interaction to.  Then we'll put it aside and build a real application, a "Create, Retrieve, Update and Delete" (CRUD) application for a simulated customer database.

# JavaFX as a Reactive Framework

This tutorial series concentrates on using JavaFX as a "Reactive" GUI framework.  

What is a Reactive framework?

From WikiPedia:

> In computing, reactive programming is a declarative programming paradigm concerned with data streams and the propagation of change...
>
> For example, in an imperative programming setting, a := b + c would mean that a is being assigned the result of b + c in the instant the expression is evaluated, and later, the values of b and c can be changed with no effect on the value of a. On the other hand, in reactive programming, the value of a is automatically updated whenever the values of b or c change, without the program having to explicitly re-execute the statement a := b + c to determine the presently assigned value of a.

In a Reactive GUI framework this usually In computing, reactive programming is a declarative programming paradigm concerned with data streams and the propagation of change. With this paradigm, it's possible to express static (e.g., arrays) or dynamic (e.g., event emitters) data streams with ease, and also communicate that an inferred dependency within the associated execution model exists, which facilitates the automatic propagation of the changed data flow.[citation needed]

For example, in an imperative programming setting, a := b + c would mean that a is being assigned the result of b + c in the instant the expression is evaluated, and later, the values of b and c can be changed with no effect on the value of a. On the other hand, in reactive programming, the value of a is automatically updated whenever the values of b or c change, without the program having to explicitly re-execute the statement a := b + c to determine the presently assigned value of a.means that the "state" of the application is expressed as data, and the GUI is linked to this data so that changes to the data are automatically reflected as changes to the GUI.  This doesn't just mean that the values displayed on the screen change, but that elements of the GUI may appear or disappear, change colours, enable or disable, animations might be triggered, or any number of elements of the user experience (UX) might react to changes in State.

Okay, but what does this get you?

In a word, simplicity.

Now your GUI is designed purely with the intent to reflect State at any given time in the best possible way.  There is no code anywhere in your GUI that concerns itself with application logic or business rules or any of that.  

At the other end, the business logic is concerned simply with maintaining and reacting to State without any idea of how it impacts the GUI.  

It's much easier to see it in practice than to describe it.  In this series, we'll start right from the beginning utilizing the tools and features in JavaFX that allow Reactive designs to be implemented.  You'll see that it's not complicated, and a natural way to build an application with JavaFX.

# Companion Project

In order to make it easier to follow along at home, all of the source code from this series has been included in a companion project available on GitHub...


[Go To the Companion Project](https://github.com/PragmaticCoding/AbsoluteBeginnersFX){: .btn .btn--info}

The source code is divided up into packages, each package containing the code as of the end the part of the series to which it corresponds.  So the package ca.pragmaticcoding.beginners.part2 contains the code as described in part2 of this series.

In this way, you can see the whole application at each stage of its evolution, you can run it for yourself and see how it works.  You can change code.  You can break code.  You can try your own ideas and experiment as much as you like.  

# On To It!

Let's get started!
