---
title:  "CSS Transitions"
date:   2024-10-08 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/css-transitions
ScreenSnap1: /assets/elements/CssTransition1.png
ScreenSnap2: /assets/elements/CssTransition2.png
ScreenSnap3: /assets/elements/CssTransition3.png
ScreenSnap4: /assets/elements/CssTransition4.png
Diagram: /assets/elements/ListProperties.png
part1: /javafx/elements/observable-classes-generics
part2: /javafx/elements/observable-classes-typed

excerpt: In JavaFX 23 we get a new feature - CSS Transitions!  Let's take a look at how this works and what you can do with it.
---


# Introduction

For me, this is the most exciting new feature added to JavaFX in version 23.  Putting transitions into the style-sheet means it will be dead simple to have `Nodes` gently fade in and fade out, resize, or change colours without writing any code at all to control it.  

The stated goal of this enhancement was to implement [W3.org CSS Transitions](https://www.w3.org/TR/css-transitions-1/) in JavaFX.  We'll take a look at that specification and see how close they got with this new feature.  

# Why Use This?

From the JDK [Issue](https://bugs.openjdk.org/browse/JDK-8311895) for this feature:

> CSS Transitions is a universally supported web standard that allows developers to specify how a CSS property changes its value smoothly from one value to another.

and a bit later:

> CSS Transitions makes it easy to define animated transitions and create rich and fluid user experiences. CSS Transitions also nicely complements control skins: while skins define the structure and semantics of a control, implicit transitions define its dynamic appearance.


First off, since these are "transitions", this is clearly aimed at elements of styling that **change**.  Secondly, this feature only applies to styling that is defined in the style sheet.  This means that styling that is applied programatically will not transition unless you explicitly program the change inside some kind of `Transition` class.  

To me, this means that this is a feature that works best with `PseudoClasses`.  As a matter of fact, the example from the JDK Issue uses the `hover PseudoClass`:

``` css
.button {
    -fx-opacity: 0.8;
    transition: -fx-opacity 1s;
}

.button:hover {
    -fx-opacity: 1.0;
}
```
The really, really nice thing about this is that you can define your `PseudoClasses` in your layout code, define conditions under which they'll turn off and on, and then you can let the stylesheet take care of the work of making the `PseudoClass` styling changes transition smoothly from one state to another.  That's a big win because all of that transitioning code just clutters up your layout code - so good riddance to it!

# An Example

Let's explore this with a simple example.  The base requirement is that we need a screen with an element that is styled with a `PseudoClass` and some way to turn the `PseudoClass` on and off:

``` kotlin
class CssTransitionExample : Application() {
    private val activatedPC = PseudoClass.getPseudoClass("activated")
    val activated: BooleanProperty = SimpleBooleanProperty(false)
    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 340.0, 200.0).apply {
            CssTransitionExample::class.java.getResource("example.css")?.toString()?.let { stylesheets += it }
        }
        stage.show()
    }

    private fun createContent(): Region = BorderPane().apply {
        center = Label("This is the Label").apply {
            styleClass += "transition-label"
            activated.subscribe { newVal -> pseudoClassStateChanged(activatedPC, newVal) }
        }
        bottom = CheckBox("Activate PseudoClass").apply {
            selectedProperty().bindBidirectional(activated)
        }
        padding = Insets(40.0)
    }
}

fun main() = Application.launch(CssTransitionExample::class.java)
```
That's all we need.  We're using a `Label` as the styled `Node` with the `PseudoClass`, and a `CheckBox` to turn the `PseudoClass` on and off.  It's pretty simple.

Here's the really cool part.  For the rest of this example, we aren't going to touch this code at all...  

Which is point of this feature.  The code just does the layout, and the necessary mechanics of handling the `PseudoClass`.  Once that's set up, everything else is done via the stylesheet.

Use a transition, don't use a transition, style it this way or some other way...none of that changes the layout code at all.

Let's look at the stylesheet without any transitions:

``` css
.transition-label {
  -fx-font-size: 18px;
  -fx-font-weight: bold;
  -fx-text-fill: black;
}

.transition-label: activated {
   -fx-text-fill: red;
}
```
That's our starting point.  Big black letters, which change to red when activated.  It looks like this:

![Screen Snap]({{page.ScreenSnap1}})

and:

![Screen Snap]({{page.ScreenSnap2}})

Now we want to turn the colour change into a transition.  All we have to do, is to change the stylesheet like this:

``` css
.transition-label {
  -fx-font-size: 16px;
  -fx-font-weight: bold;
  -fx-text-fill: black;
  transition: -fx-text-fill 5.0s;
}

.transition-label: activated {
   -fx-text-fill: red;
}
```
That's it, we just added the line `transition: -fx-text-fill 5.0s;` and now the `PseudoClass` activation will trigger a transition that will take 5 seconds for it to change the text fill from `black` to `red`.  Like this:

{% include video id="vYdvIieFuxs" provider="youtube" %}

## Some Thoughs on Transitions

This is a technical arcticle, but...

If you're building applications to be used in a professional setting, as I did for decades, then there are some things you should keep in mind.

Transitions and animations *can* make your application look slick and professional, but only if you employ them with restraint.  Screen elements that suddenly jump from colour to colour, pop in and, and suddenly change there appearance can look clumsy, and you improve this by using transitions.  But your users should never really be aware that these things are happening.

From my experience, users are conciously aware that they want functionality.  More functionality.  New functionality,  Better functionality.  If they think that you're spending time doing "fancy" stuff, they get annoyed because the perception is that the time to develop new functionality is becoming longer because of this.

On the other hand, user confidence in your application is critical.  So it does help if your application looks like it was written by programmers who know what they are doing and can craft a quality product.  Transitions can help with that.

This new feature can go a long way towards that.  How long does it take to add a single line to a stylesheet?

The 5 second transition in this example is ridiculous.  It makes the transition starkly obvious and impossible to ignore.  However, if we shorten the transition to 0.4 seconds, it changes everything:

{% include video id="Xu4KtckSjkc" provider="youtube" %}

Now the transition makes the colour change look more professional.  It's just long enough that it isn't a sharp jump from one colour to the other.

# Details About CSS Transitions

That's the basics, now let's look at some of the details:

## What Can You Transition?


## Transitioning Multiple Attributes

What if you want to transition more than a single property?  How do you do that?

It looks like you can only have a single transition per CSS selector, but you can specify more than one property for that transition.

For example, this won't work:

``` css
.transition-label {
  -fx-font-size: 16px;
  -fx-font-weight: bold;
  -fx-text-fill: black;
  transition: -fx-text-fill 0.4s;
  transition: -fx-scale-x 2.5s;
  transition: -fx-scale-y 2.5s;
}

.transition-label: activated {
   -fx-text-fill: red;
   -fx-scale-x: 2.0;
   -fx-scale-y: 2.0;
}
```
But this will:
``` css
.transition-label {
  -fx-font-size: 16px;
  -fx-font-weight: bold;
  -fx-text-fill: black;
  transition-property: -fx-text-fill, -fx-scale-x, -fx-scale-y;
  transition-duration: 3.0s;
}

.transition-label: activated {
   -fx-text-fill: red;
   -fx-scale-x: 2.0;
   -fx-scale-y: 2.0;
}
```
The implication of this is that you cannot have transitions with different durations defined.  

Also, you cannot do this:

``` css
.transition-label {
  -fx-font-size: 16px;
  -fx-font-weight: bold;
  -fx-text-fill: black;
  transition: -fx-text-fill, -fx-scale-x, -fx-scale-y 2.5s;
}

.transition-label: activated {
   -fx-text-fill: red;
   -fx-scale-x: 2.0;
   -fx-scale-y: 2.0;
}
```
Which would be nice.  However, you can do this:

``` css
.transition-label {
  -fx-font-size: 16px;
  -fx-font-weight: bold;
  -fx-text-fill: black;
  transition: all 5.0s;
}

.transition-label: activated {
   -fx-text-fill: red;
   -fx-scale-x: 2.0;
   -fx-scale-y: 2.0;
}
```

## Interpolators

Transitions do not have to move at a steady rate from beginning to end (which would be linear).  The following types of interpolation:

* Linear
* Bezier Quadratic Easing
* Step
* SMIL 3.0 Easing

The two "easing" types allow you to have the accelerate at the beggining and decelerate at the end, or both.  With the bezier quadratic easing, you can vary the amount of acceleration at each end.  

The "Step" function jumps through a number of values interpolated between the two end-point values.  You can specify how many jumps you want to have.

All of these are well documented in the [CSS Reference Guide](https://openjfx.io/javadoc/23/javafx.graphics/javafx/scene/doc-files/cssref.html#typeeasingfunction).  There are diagrams and examples.

To implement one of these, use the `transition-timing-function` tag in the stylesheet.  Like this:

``` css
.transition-label {
  -fx-font-size: 16px;
  -fx-font-weight: bold;
  -fx-text-fill: black;
  -fx-scale-x: 1.0;
  -fx-scale-y: 1.0;
  transition-property: all;
  transition-timing-function: -fx-ease-both;
  transition-duration: 2.0s;
}

.transition-label: activated {
   -fx-text-fill: red;
    -fx-scale-x: 2.0;
    -fx-scale-y: 2.0;
}
```
It's extremely easy to change the interpolator, and you really have to try them out to understand how they differ.

## Some Quirks

The example from above:

``` css
.transition-label {
  -fx-font-size: 16px;
  -fx-font-weight: bold;
  -fx-text-fill: black;
  transition: all 5.0s;
}

.transition-label: activated {
   -fx-text-fill: red;
   -fx-scale-x: 2.0;
   -fx-scale-y: 2.0;
}
```
doesn't actually work.  It transitions the scaling and the colour when you turn *off* the `PseudoClass`, but when turning on the `PseudoClass` the scaling jumps immediately, and then the colour transitions.  You can fix this by changing the stylesheet to this:

``` css
.transition-label {
  -fx-font-size: 16px;
  -fx-font-weight: bold;
  -fx-text-fill: black;
  -fx-scale-x: 1.0;
  -fx-scale-y: 1.0;
  transition: all 5.0s;
}

.transition-label: activated {
   -fx-text-fill: red;
   -fx-scale-x: 2.0;
   -fx-scale-y: 2.0;
}
```
I suspect that this might be a bug.  But it's easy enough to work around.  

I tried to get around the limitation of having only one transition per `Node` by adding a second selector to the `Label` called `growing-label` and then modify the stylesheet to use it:

``` css
.growing-label {
  -fx-scale-x: 1.0;
  -fx-scale-y: 1.0;
  transition-property: -fx-scale-x, -fx-scale-y;
  transition-duration: 26.0s;
}

.growing-label: activated {
   -fx-scale-x: 2.0;
   -fx-scale-y: 2.0;
}

.transition-label {
  -fx-font-size: 16px;
  -fx-font-weight: bold;
  -fx-text-fill: black;
  transition-property: -fx-text-fill;
  transition-duration: 5.0s;
}

.transition-label: activated {
   -fx-text-fill: red;
}
```
But this didn't work.  In fact, it turned off the transition on the scaling and then did just the transition for the colour.  

I tried this:

``` css
.transition-label {
  -fx-font-size: 16px;
  -fx-font-weight: bold;
  -fx-text-fill: black;
  -fx-scale-x: 1.0;
  -fx-scale-y: 1.0;
}

.transition-label: activated {
   -fx-text-fill: red;
    -fx-scale-x: 2.0;
    -fx-scale-y: 2.0;
    transition-property: all;
    transition-duration: 5.0s;
}
```
And the results were as expected.  The `Label` transitioned to the 2x scaling and to `red`, but snapped back to black and 1x scaling when the `PseudoClass` was turned off again.
