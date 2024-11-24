---
title:  "Why I Use Kotlin in My JavaFX Tutorials"
date:   2024-11-13 00:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/why_kotlin
Diagram: /assets/posts/MVCI.png
Dispatch1: /assets/elements/EventDispatchChain1.png
Dispatch2: /assets/elements/EventDispatchChain2.png
DemoRunning: /assets/posts/MvciDemoRunning.png
DemoDone: /assets/posts/MvciDemoDone.png
Chain1: /assets/elements/Chain1.png
Chain2: /assets/elements/Chain2.png
excerpt: Not everyone can understand Kotlin code.  So why use it in my JavaFX tutorials?  Here's why.
---

# Introduction

I realize that the vast majority of people who read my articles are not coding in Kotlin, have never coded in Kotlin and probably have only a limited understanding, if any, of Kotlin.  Does this turn people off?  

Maybe.  I received this feedback on-line:

>  I don't understand where you get the idea that every Java developer understands or likes Kotlin code. I see Kotlin code, I close the tab, sorry.

So why write the examples in my tutorials in Kotlin?  There are a number of reasons...

# Kotlin is Just Better

Kotlin is everything that Java wants to be, but never will be.  There really is no other way to characterize it.

When I switch back to Java, my first thought is, "This is so clunky!".  Of course it's not.  It's the same Java that it's always been, it's just that the clumsy stuff I never noticed before can't be unnoticed after months and month of Kotlin.

# Kotlin Means Less Code

I struggle to keep these tutorials down to a reasonable size.  

There's often a lot of content to cover, and can take lots of examples and progressive elaboration of the concepts to present the concepts in a manner that I think will be clear and easy to understand.  

Any bloat that I can trim out is a win in my books, and Kotlin helps trim the bloat out of the code examples.  The result, I think, lets the ideas come through without forcing the reader to wade through extra code that would get in the way. Here's a tiny snippet:

``` kotlin
private val label = Label(labelText).apply {
    layoutX = 10.0
    styleClass += "pane-label"
    widthProperty().addListener(InvalidationListener { setClipping() })
    heightProperty().addListener(InvalidationListener {
        setClipping()
    })
    maxWidthProperty().bind(this@LabelledPane.widthProperty().subtract(24.0))
    isWrapText = true
}
```

First off, because of the `apply{}` scope function, which passes the result of `Label()` into the function as `this`, we can actually do all of the set-up for the field in one place. So you don't have to go into constructor code to see how the field `label` is initialized.

Secondly, in Java every single line of code that in Kotlin is inside `apply{}` would need to start with `label.`, which just clutters stuff up. In Kotlin, `label` is `this` inside of `label.apply{}`, and `this` can be inferred (just like in Java) when it's clear. Not to mention the lack of semi-colons at the end of every line.

Thirdly, in Kotlin we have field accessors which execute the setters and getters automatically. So this line in Java:

``` java
label.getStyleClass().add("pane-label");
```

becomes
``` kotlin
styleClass += "pane-label"
```

In my opinion, the block of code above is super easy to read. At a glance, you can see that we are creating a field called label, and we're moving it, styling it, putting Listeners on the width and height and wrapping the text.

Is it hard for Java only coders to understand? I don't think it should be. IMHO there's a world of difference between reading and getting the gist of code versus writing valid code. The level of knowledge for simply understanding code is so much lower that virtually any Java programmer should be able to look at the Kotlin code and at least get the idea about what's going on.

I mean, you don't have to understand about field accessors to intuitively understand that `layoutX = 10.0` is somehow the same as `label.setLayoutX(10.0)`. The important point is that `layoutX` is getting set. And these articles are supposed to be about understanding how to deal with concepts in JavaFX, not about being a source for copy/paste into your own projects (which I'm totally fine with people doing, BTW).

I'll also point out that I avoid using the full toolset of utilities that I've written for my own personal projects that really, really squeeze the rubbish boilerplate out of layout code. Something like this:

``` kotlin
children += promptOf(set.setNum.asString()) setSize setNumSize aligned Pos.CENTER
```

Which is pretty straight-forward to Kotlin coders but would be probably just too baffling to Java coders.

# Kotlin Makes JavaFX Better

This is really the biggest thing for me.  Tons and tons of JavaFX code involves configuring `Node` subclasses.  This usually involves, in Java, in instantiating the `Node` as a variable, calling a few methods of the `Node` to add styling, to bind a value, to configure alignment, to add an `EventHandler`, or whatever you need to configure.  Then add the `Node` to the layout.  

In Kotlin, all that code is so much cleaner with a Scope Function like `apply{}`.  You don't need to instantiate the `Node` as a variable, just call the constructor and the add `.apply{}` at the end.  And inside the `apply{}`, the `Node` becomes `this`.  And `this` can be inferred when it's clear - so you don't even need it.  

Oh!  The tedium of `pane.getChildren().addAll()`!  In Kotlin, inside `.apply{}` it's just `children += `.

Writing a builder/helper method for a `Node` is simple too.  All it takes is `fun buildNode():Node = SomeNode().apply{}`.

The result, in my opinion, is example code where the central ideas are much more clear than they would be with Java.

# Kotlin Fixes Stuff

The easiest example to think about here is "Null Safety".  Theoretically, this dealt with in Java - to a large extent - with the introduction of `Optional`.  But how many people actually use `Optional`.  In Kotlin, variables **CANNOT** be null unless you specify them as "nullable".  This means putting a "?" at the end of the type.  

For instance, `Int` cannot have a null value, but `Int?` can.  Take a look at this invalid code:

``` kotlin
var a :Int? = 5
var b :Int = a + 3
```
This will generate a compiler error.  

You can't perform a mathematical function on `Int?`.  It's not an integer, it's a "nullable integer".  You can, however, do this:

``` kotlin
var a :Int? = 5
var b = a?.plus(2)
```
That compiles, because the null safety operator, `?` will return `Null` if the value of `a` is `Null`, will perform the function call if it's not `Null`.  But now `b` is also an `Int?`, not an `Int`.  If you want `b` to be `Int` then you have to tell the assignment what to do in the case that `a` is `Null` using the "Elvis Operator":

``` kotlin
var a :Int? = 5
var b = a?.plus(2)?:0
```
Now, `b` is an `Int`, and will have the value `0` when `a` is `Null`.

In Kotlin, use of these constructs in not optional, unlike `Optional` in Java which is completely optional to use.  If you want a variable to be declared but initialized later, then it has to be nullable.  If it's nullable, then you need to use the null safety operator to deal with it.

On top of all of that, even though it completely changes the way that you think about `Null`, it's so much cleaner and easier to us than `Optional`.  The null operator works brilliantly with Scope Functions and everything just fits together nicely.

# Spreading the Kotlin Word

## Kotlin as a JavaFX Layout DSL

I've always been clear that I think that FXML is a waste of time.  It does allow you to use SceneBuilder, but I don't see a great deal of value in that either.  Instead, I've argued that well designed hand-written layout code is far easier to deal with - meaning to write, understand and maintain - than FXML and the complications that it brings.  

I've also argued that the strict application of just two "Clean Coding" rules - "Don't Repeat Yourself", and the "Single Responsibility Principle" - will inevitably result in the development of a library of builder or "helper" functions that you can use over and over again.  This will leave your actual layout code lean and clean.

Kotlin let's you take this one step further.  The use of `.apply{}` allows you to implement any number and combination of internal `Node` configuration methods into "decorators" that can be chained together.  The `infix` qualifier allows you to strip away the `.()` part of the call.  Extension functions allow you to add new methods to `Node` classes that, in turn, can be used as decorators if you want.



# Understanding Kotlin Code

Here's an example of some Kotlin code from another tutorial:

``` kotlin

```
We'll refer back to this code in the items below...



## Basic Improvements

You'll notice that the example code has no semicolons.  Yeah!  Who needs them?  They're not illegal, but why bother with them.

You'll also notice that constructors are called just by using `ClassName()` without `new`.  Yeah, again!  

This follows a basic principle in Kotlin that useless stuff that doesn't add any value isn't included.

## Scope Functions



## Fields vs Properties

## Accessors

Now that you understand that fields in Java are replaced by properties in Kotlin, and that they have "baked in" getters and setters, it's easy to understand why Kotlin essentially does away with calls to the getters and setters.  The preferred way is to use what is called the "accessors", which look like a direct reference to the property, but actually invoke the getters and setters attached to the properties.

What's especially cool, though, is how Kotlin interacts with Java classes.  When Kotlin sees a getter or setter in Java code, it let's you use an accessor to call the getter or setter!  
