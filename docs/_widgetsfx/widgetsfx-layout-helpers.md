---
title:  "WidgetsFX -Layout Helpers"
date:   2024-01-05 12:00:00 -0500
categories: widgetsfx
logo: /assets/logos/JavaFXLogo.png
permalink: /widgetsfx/layout-helpers
DocsPage: /widgetsfx-docs
ScreenSnap1: /assets/elements/ListView1.png
ScreenSnap2: /assets/elements/ListView2.png
ScreenSnap3: /assets/elements/ListView3.png
ScreenSnap4: /assets/elements/ListView4.png
ScreenSnap5: /assets/elements/ListView5.png

excerpt: Introducing 
---

# What are Layout Helpers?

Standard JavaFX layout coding in Java suffers from the following problem:

``` java
   .
   .
   .
   Label namePromptLabel = new Label("Name:");
   namePromptLabel.getStyleClass().add("prompt-label");
   TextField nameTextField = new TextField();
   nameTextField.getStyleClass().add("input-field");
   nameTextField.textProperty().bindBidirectional(model.name);
   HBox container = new HBox(3, namePromptLabel, nameTextField);
   .
   .
   .
```

And if you have a lot of `Label`/`TextField` pairs in your layout, then you'll be doing this a lot.  That is, unless you apply DRY (Don't Repeat Yourself) and then you'll extract some methods to eliminate the repetition.

Something like this:

``` java
private Label promptLabel(String text) {
    Label result = new Label(text);
    result.getStyleClass().add("prompt-label");
    return result;
}

private TextField inputField(StringProperty boundProperty) {
   TextField result= new TextField();
   result.getStyleClass().add("input-field");
   result.textProperty().bindBidirectional(boundProperty); 
   return result;
}
```

And now your layout code can look like this:

``` java
HBox container = new HBox(3, promptLabel("Name:"), inputField(model.name));
```

It's clear that this is much more readable than the first version.  

But what if it's a "one-off" and DRY isn't going to apply?  Surely that new, one line, version is still better?

Well, we still have the other key principle to follow when writing layout code, the "Single Responsibility Principle", and it can help us.  There's actually two things going on in our first version: layout and configuration.  That's what leads to our more complicated code.  So you still want to extract out the configuration part and put it into a builder/factory method and get it out of your layout code.

As much as possible, that's what you want your layout code to be - layout code.  And that's where the layout helpers in WidgetsFX come in.

The truth is that all of that configuration code is pretty routine and uninteresting, it really is just boilerplate - a form you have to go through to get the result you want.  If you can get it out of your layout code and have it somewhere where it does its job and works properly so that you don't have to think about it or test it, then that's a big win.

## Less Code is Better Code

Generally speaking, the less code that you have the easier it will be to read, understand, maintain and enhance.  This doesn't mean code that's been made ridiculously terse or over-abstracted with 1 letter names for things.  But well written code that does things in the simplest way possible.  The layout helpers in WidgetsFX are designed to reduce the amount of code, especially boilerplate, that you need to get things done.

## Layout Code Should be Layout

There is layout, and there is configuration.  When you are looking at layout code, your first goal is to understand how that code relates to the GUI that you see on the screen.  Including configuration code in with your layout code just makes it more difficult to follow the design of the screen.  Very often, you aren't interested in *how* a `Node` is configured, but you do care about *what* it's configured as.  The layout helpers in WidgetsFX are designed to get that configuration out of your layout code while having names that tell the programmer about how it has been configured.

## Some Stuff You Do a Lot

Everyone has a different style and a different approach to building GUI's, but regardless of this there are going to be certain things that every programmer finds themselves doing over and over again.  Certain styleclass selectors are going to get applied to certain `Node` classes repeatedly across one or many layouts, and certain operations get used frequently.  The layout helpers in WidgetsFX provide some of these, that you can adapt to your own use the way that you like to create layouts.

# Configuration Via Decorators

Kotlin makes it super easy to define a decorator via "Extension Functions" that use the `apply{}` reciever method.  What this means is that functions can be added to the standard JavaFX `Node` subclasses that return that `Node` as the result.  For instance:

``` kotlin
fun Node.styleAs(selector: String): Node = apply{
    styleClass += selector
}
```
Which means that you can do something like this:

``` kotlin 
borderPane.setCenter(HBox(5.0, label1, label2).styleAs("red-box"))
```
Furthermore, Kotlin also allows certain functions that take only a single parameter to be defined as `infix`.  This means that you can drop the `.` and the enclosing `()` around the parameter.  This would yield:

``` kotlin 
borderPane.setCenter(HBox(5.0, label1, label2) styleAs "red-box")
```
In certain situations, the infix notation can be much cleaner and more intuitive to read than a standard decorator call.  Especially when multiple decorators are chained together.  If you don't like it, the infix notation is always optional.

# Regions, Nodes and Scenes
WidgetsFX has layout helpers that work for all `Nodes` or `Regions`:

## Testing Styles
At times, it can be difficult when designing a layout to understand how `Regions` or `Nodes` are actually contained and fill the space on the screen.  For this purpose a set of test styles have been created that add a border and a background colour to whatever `Node` they are applied to.  Using different styles on different container classes in your layout can quickly reveal why your layout isn't behaving as you thought it should.

Three styles are defined `RED`, `GREEN` and `BLUE`.  The function to apply them is `Node.testStyleAs()`, this is also available as infix:

```kotlin 
  val hBox1 = HBox(10.0, label1, textField1).testStyleAs(TestStyle.RED)
  val hBox2 = HBox(10.0, label1, textField1) testeStyleAs TestStyle.RED
```

## Adding a PseudoClass 

If you read the official JavaDocs, you'll think that adding a pseudoclass to `Node` is a big complicated thing.  It doesn't have to be.

The most complicated part is converting your BooleanProperty that controls the pseudoclass to the action that flips the pseudoclass state.  WidgetsFX contains a function called `Node.bindPseudoClass()` that handles all the work for you.  You need to supply the PseudoClass, but WidgetsFX will put a listener on it for you.

## Controlling Visibility

One of the big "gotchas" with JavaFX for most beginners is that making a `Node` invisible doesn't mean that it won't still take up space on your layout!  To make it disappear without a trace, you need to turn off its `Managed` property as well.  Then you have lots of code that has to deal with both `Visible` and `Managed`.  WidgetsFX handles that for you.

* Node.isHidden()<br>
This function accepts a `Boolean` and flips both `Node` properties for you.  Available as infix.
* Node.bindHidden()<br>
This function accepts an `ObservableBooleanValue` and binds both `Node` properties for you.  Available as infix.
* Node.visibilityOf()<br>
Sometimes you do just want to make a `Node` invisible while still taking up space in your layout.  This function provides an infix decorator function to do this.

## Insets

How much does your code get clogged up with `node.setPadding(new Insets(6.0))`?  How often do you just want the same padding on each side of your `Region`?

Introducing `Region.padWith()`.  And it just takes a `Double`!  And it's infix!  

Now you can just go `HBox() padWith 7.0`.

## Decorators for Standard Configurations 

The biggest contributor to excessive boilerplate in JavaFX is the way that `Nodes` are configured.  In Java, you have to instantiate the `Node` as a variable, add lines of code to call the various configuration methods on that variable, and then insert the variable into your layout.  Yuk!!!

Even in Kotlin, this still requires using `Node.apply{}` and lines of code to call the configuration methods.  It avoids the need to intstantiate a variable, but still looks ugly in your layout code.

WidgetsFX provides a number of standard configuration methods as infix decorator functions.  The result is a very concise, readable format that doesn't clutter up your layout code:

* Region.minWidthOf()<br>
Sets the minimum width of a `Region`.  Available as infix.
* Region.maxWidthOf()<br>
Sets the maximum width of a `Region`.  Available as infix.
* Region.minHeightOf()<br>
Sets the minimum height of a `Region`.  Available as infix.
* Region.maxHeightOf()<br>
Sets the maximum height of a `Region`.  Available as infix.
* Region.prefWidthOf()<br>
Sets the preferred width of a `Region`.  Available as infix.
* Region.prefHeightOf()<br>
Sets the preferred height of a `Region`.  Available as infix.
* HBox.alignTo() and VBox.alignTo()<br>
Sets the alignment of a VBox or HBox to the specified `Pos` value.  Available as infix.



# Labels

## Builders/Factory Methods

WidgetsFX defines a number of standard `Label` stylings, provided via stylesheet selectors which are applied through builders which will also provide bindings to `ObservableStringValues` if you want.  So in a single call, you can instantiate a `Label` styled in a particular way and bound to an external property.

There are three heading styles, `H1`, `H2` and `H3`.  Created through `h1Of()`, `h2Of()` and `h3Of()`.  All of these builders support a fixed text value or bound StringProperty and, optionally, a graphic `Node`.

There is a style called `PROMPT` (it would have been nice to call it "LABEL", but that would have caused confusion).  This style is intended to be used for `Labels` that sit beside input controls, or other `Labels` that display data.  Use the method `promptOf()`

There is a style called `DATA`, created via `dataOf()`.  This is intended to be used for places where you wish to display data, as opposed to static elements of your screen.

## Configuration Decorators 

WidgetsFX has decorators that cover most of the configuration that you're likely to use with `Label`:

* bindTo()<br>
Binds the `TextProperty` of the `Label` to another `StringProperty`.  
* bindGraphic()<br>
Binds the `GraphicProperty` of the `Label` to an `ObservableObjectValue<Node>`.
* oriented()<br>
Sets the position of the `Graphic` relative to the `Text`.  This is equivalent to setting `ContentDisplay` on the `Label`.  This is available as infix.
* styleAs()<br>
Applies one of the standard `LabelStyle` selectors to the `Label`.  This is available as infix.   
* wrapText()<br>
Turns the text wrapping feature on.
* underlined()<br>
Sets the underlining of the text on or off.  This is available as infix.
* plusAssign()<br>
This is an operator function which will bound the `StringProperty` on the right side of `+=` to the `TextProperty` of the `Label`





