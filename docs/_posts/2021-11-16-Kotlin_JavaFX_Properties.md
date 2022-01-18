---
title:  "Kotlin: JavaFX Properties"
date:   2021-08-24 12:00:00 -0500
categories: javafx kotlin
logo: /assets/logos/Kotlin.png
permalink: /javafx/kotlin_properties
excerpt: An approach to implementing the JavaFX Property "bean" structure for observable objects in an idiomatic Kotlin fashion.
---

# JavaFX Properties

In JavaFX, properties and other observable classes are used to implement reactive programming techniques - which is absolutely the best way to design a JavaFX application.  Ideally, you should implement your data elements as properties and then use the Binding library to connect the data to your GUI.  

## JavaFX Properties in Java

In Java, JavaFX properties should be placed into a Model with a "bean" structure.  This means that you should have the following 3 methods in your Model for each property that you define:

1. A delegate getter for the property value
1. A delegate setter for the property value
1. A getter for the property itself

There is a naming convention that you should follow, and you can see it in the code snippet below:

{% gist 3a2e8ae215eaee024386a9867a5e67f1 %}

Perhaps the most important part is that the getter for the property should have the name "{Property Name}Property".  There are some built-in parts of JavaFX which will use reflection to access the property and will need need this structure.

if you are using Intellij Idea, it will create the three methods for you once it's identified the lonely field declaration as an issue.

### The Property is Final

Notice that the property in the example is has the "final" qualifier.  This is because the property itself, viewed as a container, should never be changed.  The contents can change - that's the point - but you should never change the reference of the property itself or you'll lose any bindings or listeners on the property.  

## Kotlin Getters and Setters

Somewhat confusingly for JavaFX programmers, Kotlin calls class fields (or "instance variables"), "properties".  This isn't technically precise, but if you want to know more you can read the  [Official Kotin Page](https://kotlinlang.org/docs/properties.html) about it.  Properties in Kotlin are wrappers for fields which are hidden from direct access with default getters and setters each property.  Kotlin syntax allows the calls to the getters and setters to look like direct references to the properties.  Like this:

{% gist e7e3c38f85227842558e3ebc79f6c9ff %}

It's possible to override the default getter and setter for a property, as in this code:

{% gist 34b03c5530a07313dca2f4c27b4c5785 %}

If you run this code, you'll see the following output:

```
in the setter
in the getter
Nickname: Shorty
```
So you can see that, although it looks like you're referencing the field directly, you are actually calling the getter and the setter defined for the property.

Perhaps even more importantly, though, is that you can access the getter and setter from Java through calls to `testClass.setNickName()` and `testClass.getNickName()`.  Kotlin automatically handles this for you.  By the same token, you can access getters and setters in Java classes from Kotlin using direct references to the fields.

## The Goal for JavaFX Properties in Kotlin

When creating JavaFX properties in Kotlin, we want them to look externally exactly like they would in Java.  Meaning that they have to have the same JavaFX "bean" structure with the same method names.  Of course, in Kotlin the getter and setter need to still work same way that they would for any other Kotlin property - looking like direct access.

In this case, however, the Kotlin approach makes things appear to be a little more complex.

### Dealing With the "Final" Qualifier

In Kotlin, variables can either be immutable or mutable, which is set by the use of the "val" or "var" keywords when the variable is defined.  Immutable variables, defined with the "val" keyword cannot have setters.  This is an issue because, while you can always override the default getter to delegate to the property `getValue()` method, you can't even define a setter for a "val" variable.

This makes perfect sense, but it means that we need to use a slightly roundabout route to implement a JavaFX "bean".

## An Approach That Works

A good answer seems to be a fairly common design pattern in Kotlin, and that's to use a hidden backing property.  In this case, the hidden backing property is the actual JavaFX property type that we are going to use.  There's also a public property, which has the same type as the data contained in the JavaFX property, and which delegates it's getters and setters to the hidden properties, `get()` and `set()` methods.  

Finally, a method is added to the Model to return the JavaFX property itself, using the same naming convention that you would use in Java.

The effect is that the publicly visible property will never actually have any data assigned to it, since it's setter is overridden to put the data into the hidden backing JavaFX property.  It only exists to present a public interface to the methods in the JavaFX property.

The result looks like this:

{% gist f6d3bd9dfc261b6d411d7945172d5833 %}

And the output looks like this:

```
Nickname: Shorty
Property: ObjectProperty [value: Shorty]
```

It seems to be the convention in the examples I've seen to use "_" prefix for hidden properties, as seen in that same official Kotlin page - in the section on  [Backing Properties](https://kotlinlang.org/docs/properties.html#backing-properties) .  So I've followed that here.

You should also note that, even though it seems quite verbose, this is actually a little less code than you'd need to implement a JavaFX bean in Java.  Which is pretty normal for Kotlin.

## An Added Bonus

If you're using Intellij Idea, like I do, then you might find the following like template useful:

{% gist d301cb308d2b4741220c6146cf3ba69c %}

Just give it a short name (I use "fxprop") and it'll make your life a lot easier.

I've only implemented it as `ObjectProperty<{Type}>` because that works best.  `StringProperty` might be useful, but the `Number` based properties like `IntegerProperty` and `DoubleProperty` have type issues when you get into complicated stuff.  So they seem easier to use, but `ObjectProperty<Int>` is almost always a better choice than `IntegerProperty``.
