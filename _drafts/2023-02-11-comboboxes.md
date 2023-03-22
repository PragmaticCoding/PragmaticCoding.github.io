---
title:  "JavaFX: Understanding the ComboBox"
date:   2023-02-11 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/comboboxes
ScreenSnap1: /assets/posts/StarterFX.png
ScreenSnap2: /assets/posts/StarterFX.png
ScreenSnap3: /assets/posts/StarterFX.png

excerpt: Taking a look at the structure of the JavaFX ComboBox, and how to do cool things with it.
---

# Introduction

The [ComobBox](https://openjfx.io/javadoc/16/javafx.controls/javafx/scene/control/ComboBox.html)!  Everybody's go-to control for multiple choice input.  It's one of those elements that's dead simple to get started with, but has a lot of power that many people don't use.

# Simple Implementation

The easiest way to use a `ComboBox` is to just stuff some values in a list, bind the `ValueProperty` and stick it in a layout:

``` java

```
It starts out with a `Null` value, so the `ComboBox` just looks like an empty `Button` with an arrow.  Click anywhere on it and it opens up with a list of choices for the user to click on.  Picking one then populates the `Button`, and closes the pop-up list.

# Standard Configuration Options

Editable
: ComboBoxes work in two modes, editable and non-editable.  "Editable" means that the user can type into the area in the ComboBox that looks like a `TextField` and enter values that aren't in the pop-up list.  The default is non-editable.

Items
: The contents of of the pop-up list are controlled by an `ObservableList` called `items`.

VisibleRowCount
: This is the maximum size of the pop-up list, in rows.  The default is 10, but you can make it bigger or smaller, depending on your layout.

Value
: The currently selected value in the `ComboBox`.  This can be set programmatically if, for instance, you want to set a default value.  Note that if you do change the value programmatically, it will automatically select the corresponding value in the pop-up list.  The value does NOT have to be one of the values in the pop-up list, however.

OnAction
: You can trigger an `EventHandler` whenever the `ValueProperty` of the `ComboBox` is changed.  This is the `OnAction` event.

The pop-up list works very much like a `ListView` and does, therefore, have a `SelectionModel`.  For some reason, and I don't know why (probably some copypasta that ended up in every on-line tutorial), people seem to think that the way to get a `ComboBoxes` value is to go through the pop-up list `SelectionModel`.  In reality, there's almost never any reason to go through this round-about route - just use the `Value` property.

# ComboBox<Object>

So what if you don't want to have your `ComboBox` return a primitive like `int`, or a `String`? How do you do that?

First off, you need to remember that the items that populate the list in a `ComboBox` are part of your Presentation Model, so they should be designed with the intent that they are going to be used to support your view.  So the idea that you're just going to rip some customer records out of a database and dump them into a `ComboBox` as the choices is probably a bad design decision.  

That being said, there are times when you might have a more complex object used to populate your `ComboBox`.  It's generally considered bad form to rely on `toString()` to create the display for your `ComboBox`, so you'll need to customize it in some way to make it work properly.  

In this example, we're going to look at using an object with two fields.  This will be a "Customer" record with an account number and a name.  We'll need the account number to look the customer up in the business logic, but the users need the name to select which customer from the `ComboBox`...



# Linking ComboBoxes

Something that comes up fairly often is the idea of having two `ComboBoxes` that are somehow connected to each other.  Changing the selection in one `ComboBox` changes the list of items available in the second `ComboBox`.
