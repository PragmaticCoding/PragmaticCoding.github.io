---
title:  "WidgetsFX - A Library For JavaFX Development"
date:   2024-01-05 12:00:00 -0500
categories: widgetsfx
logo: /assets/logos/JavaFXLogo.png
permalink: /widgetsfx/main
DocsPage: /widgetsfx-docs
ScreenSnap1: /assets/elements/ListView1.png
ScreenSnap2: /assets/elements/ListView2.png
ScreenSnap3: /assets/elements/ListView3.png
ScreenSnap4: /assets/elements/ListView4.png
ScreenSnap5: /assets/elements/ListView5.png

excerpt: A Kotlin-based library of factory methods, builders, Properties and custom controls to make JavaFX development easier.
---

# Why WidgetsFX?

If you make the very sensible decision to avoid FXML and SceneBuilder, instead coding your layouts by hand you will find that the two key principles that you need to follow to keep your layout code concise and clear are DRY (Don't Repeat Yourself) and SRP (the Single Responsibility Principle).  Of the two of these, DRY is particularly important.

If you are following DRY, you will undoubtedly find yourself creating a library of factory methods that do the kind of things that you commonly do with layouts.  This is inevitible because as soon as you find yourself duplicating some code in a layout, you'll pull it out into its own method and when you find yourself writing the same method in several layouts, you'll pull it out into a common class somewhere else in your project.

## Layout Helpers

One of the biggest complaints about JavaFX is that it's heavy on the "boilerplate" code.  This is valid, but also easy to fix by creating factory methods and Kotlin extension functions that automate some of the boilerplate.

## DirtyFX

If you are building a CRUD style system with a Reactive design, then you'll eventually ask yourself, "How can I tell when the user has updated the screen?".  DirtyFX supplies a set of Property classes that keep track of a baseline value and can contain a `BooleanProperty` that will tell you if the current value is different from the baseline value.

## Custom Components

The set of standard components and controls in JavaFX is fairly comprehensive, but JavaFX also gives you access to the raw building blocks to create your own components that work just as well as the built-in ones.  WidgetsFX contains a library of custom components that you might find useful in your own projects.

# Kotlin 

If you have enough control over your projects and the environments that you work in to use Kotlin...do it!

Kotlin is everything that Java wants to be but doesn't know it yet.

Seriously, Kotlin + JavaFX is a match made in heaven.  So much of the boilerplate and verbosity of JavaFX layout code can simply be avoided by using Kotlin's extension functions, operator functions and infix functions.  Well written JavaFX layouts written in Kotlin that take advantage of these features looks more like a scripting language for layouts than they do like a general purpose programming language.

WidgetsFX will work with Java, but it really shines when used with Kotlin because all the infix and extension functions merge seemlessly with layout code in Kotlin.

# What About TornadoFX?

The approach taken by TornadoFX is to use configurable builders to do all the work.  You write your layout code by instantiating the builders and then configuring them, adding builders to builders and configuring them and so on.  If there's a pattern that *you* use all the time, then you'll need to write a custom library method or class that executes that pattern in the TornadoFX builders.

In the end, you've injected a new layer into the process and the resulting code looks much, much simpler than pure Java if you don't use DRY and SRP.

However, you don't need all that stuff to simplify your layout code - especially if you're writing in Kotlin.

The approach taken in WidgetsFX is different.  Sure, there are *some* builders, but they are mostly simple and designed to instantiate a very spceifically configured `Node`.  Builders like `promptOf()`, which creates a `Label`, potentially bound to `Property`, styled in a particular way.  

Some builders in WidgetsFX simply supply the equivalent of constructors that JavaFX should have included but didn't.  Like `labelOf(value: StringProperty)` which creates a `Label` with its `TextProperty` bound to the supplied `StringProperty`.  This is essentially the equivalent of `new Label(StringProperty value)` which should have been (but wasn't) included in JavaFX.

You can use as much or as little of the builders and functions in WidgetsFX that you like.  Use the infix decorators as infix, or as regular decorator functions - whichever looks clearest to you.  There's no need to learn a whole new way of building layouts, take your own approach and let the utilities in WidgetsFX get the repeated drudgery out of your layouts.


# Documentation

The KDoc API documentation can be found [here]({{page.DocsPage}})

