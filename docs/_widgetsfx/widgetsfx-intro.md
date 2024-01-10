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

# Kotlin 

If you have enough control over your projects and the environments that you work in to use Kotlin...do it!

Kotlin is everything that Java wants to be but doesn't know it yet.

Seriously, Kotlin + JavaFX is a match made in heaven.  So much of the boilerplate and verbosity of JavaFX layout code can simply be avoided by using Kotlin's extension functions, operator functions and infix functions.  Well written JavaFX layouts written in Kotlin that take advantage of these features looks more like a scripting language for layouts than they do like a general purpose programming language.

# Documentation

The KDoc API documentation can be found [here]({{page.DocsPage}})

