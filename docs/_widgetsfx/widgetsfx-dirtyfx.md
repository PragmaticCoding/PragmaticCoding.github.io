---
title:  "WidgetsFX: DirtyFX Properties"
date:   2024-01-05 12:00:00 -0500
categories: widgetsfx
logo: /assets/logos/JavaFXLogo.png
permalink: /widgetsfx/dirtyfx
DocsPage: /widgetsfx-docs
ScreenSnap1: /assets/elements/ListView1.png
ScreenSnap2: /assets/elements/ListView2.png
ScreenSnap3: /assets/elements/ListView3.png
ScreenSnap4: /assets/elements/ListView4.png
ScreenSnap5: /assets/elements/ListView5.png

excerpt: Introducing 
---

# What Are DirtyFX Properties

If you are using JavaFX as a Reactive platform, you'll have the "State" of your GUI represented by a Presentation Model that's bound to the properties of your GUI `Nodes` and shared with your business logic.  In that Presentation Model, you'll have `ObservableValue` and `Properties` that hold the current value of the data values held in your screen.  In most CRUD-like applications, you'll have a `Button` that says, "Save" or "Update" or something like that.

But how do you know if the data on the screen and in the Presentation Model has changed, and that an update is required?

And that's where the DirtyFX `Properties` come in.  DirtFX properties support the idea of a "baseline" value for a `Property`, and when the current value of the `Property` different from the baseline value, then the `Property` is considered "dirty".  DirtyFX `Properties` also support the concept of resetting the current value back to the baseline value and, conversely, setting a new baseline value that is the same as the current value.

# A Little History 

The original implementation of DirtyFX comes from Thomas Nield's excellent [DirtyFX](https://github.com/thomasnield/DirtyFX) library published in 2018.  He's indicated in his GitHub account that he's too busy to maintain most of his libraries, so feel free to fork them and support them.  So here we are.

The original DirtyFX library was also written in Kotlin, but the style is not very Kotlin-like (IMHO).  For most of the classes, he's simply used composition to build a wrapper around two `Simple{Type}Property's` (one for the current, and one for the baseline value), and the delegated all the normal `Property` functions that you might call to the one that holds the current value.

However, when you look at the source code for a class like `SimpleStringProperty`, you find that it extends the abstract class `StringPropertyBase` and adds only the functionality to support the two methods not implemented in `StringPropertyBase` that are declared in abstract classes further up the hierarchy.  These methods are `getBean()`, and `getName()`.  `SimpleStringProperty` supplies the fields to hold these values, and implements the constructors to populate them.  All the other `Simple{Type}Properties` are the same.

It seemed better to me to replicate this functionality and incorporate it in the DirtyFX `Properties` directly, and leave the Simple{Type}Properties out the mix entirely.  Kotlin supports abstract fields and default methods in `interfaces`, so most of the mechanics can be handled inside of the `DirtyProperty` interface.  

The result is that each `Dirty{Type}Property` class is only about 20 lines long, most of that devoted to handling `getName()` and `getBean()` and the associated constructors.  

I felt that Thomas' `rebaseline()` method name was needlessly long, so I've used `rebase()` instead.  Otherwise, the usages are the same here as in the original DirtyFX library.

Finally, I've never been able to see a need or decide how `DirtyCollectionProperties` should work.  So I've left them out of the WidgetsFX implementation of DirtyFX.  If you need them, then you should probably use Thomas' implementation.

# Potential Overhead of the DirtyFX Properties

The DirtyFX `Properties` replace `Simple{Type}Properties` and their base operation works identically to the `Simple{Type}Properties` as most of the functionality is handled be `{Type}PropertyBase`.  However, the base value is held in an `ObjectProperty<{Type}>` field of its own, and the `isDirty` field is held as a `Binding` between the current value and the base value.

In most applications, you'll never see a performance difference, and there would be no problem with just using `Dirty{Type}Property` instead of `Simple{Type}Property` everywhere.

# The Classes

The following classes are supplied:

1. DirtyStringProperty
1. DirtyBooleanProperty
1. DirtyIntegerProperty
1. DirtyDoubleProperty
1. DirtyObjectProperty
1. DirtyLongProperty
1. DirtyCompositeProperty

Except for `DirtyCompositeProperty` these all work the same as the `Simple{Type}Properties` but support `reset()`, `rebase()` and `isDirtyProperty()`.

# CompositeDirtyProperty 

`CompositeDirtyProperty` is an agregator class for DirtyFX `Properties`.  

