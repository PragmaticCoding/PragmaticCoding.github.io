---
title:  "WidgetsFX - Custom Controls & Components"
date:   2024-01-05 12:00:00 -0500
categories: widgetsfx
logo: /assets/logos/JavaFXLogo.png
permalink: /widgetsfx/controls
DocsPage: /widgetsfx-docs
ScreenSnap1: /assets/elements/ListView1.png
ScreenSnap2: /assets/elements/ListView2.png
ScreenSnap3: /assets/elements/ListView3.png
ScreenSnap4: /assets/elements/ListView4.png
ScreenSnap5: /assets/elements/ListView5.png

excerpt: Custom controls and components for JavaFX layouts
---

# Custom Components

# The Components

## TwoColumnGridPane

`GridPane` is probably the second most over-used and abused of the JavaFX layout classes.  Beginners seem to love the idea shoving everything into a row/column format and it somehow satifies their need for order - I guess.  But `GridPane` can become really tedious to use, everything has to be put into an exact row and column, it's difficult to insert a row in the middle of a `GridPane`, and the row and column constraints can be difficult to work with.  Most of the time, you can get better, simpler results by nesting `HBox` and `VBox` regions.  But if you really do want things to line up exactly into columns and rows, then `GridPane` can be the right layout to use.

One use case that comes up quite often is a situation where you want to have part of the screen laid out with a column of `Labels` or "prompts" and then a column of `TextFields` or data display `Labels` lining up just to the right of them.  Two neat columns, the left column right justified and the right column left justified, so that everything lines up nicely down the middle. 

And this is `TwoColumnGridPane`, a custom class that's really nothing more than a standard `GridPane` formatted up properly and loaded up with a set of methods to easily allow you to add [prompt, data] pairs into the `GridPane` without have to worry about counting rows.

## InputActionWidget

There's lots of occasions when you need the combination of a `Label`, a `TextField` and a `Button` to perform some function in a screen.

## RadialMenu