---
layout: single
title: All About TextFormatter
toc: true
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
permalink: /javafx/elements/textformatter/

feature_row:
  - image_path: /assets/images/LittleBrain.jpg
    alt: "placeholder image 1"
    title: "Part 1"
    excerpt: "In this article you'll be introduced to the idea of a `TextFormatter`.  You'll learn how to create a `Filter` and a `Converter` for a `TextField` so that it will only accept numbers."
    url: "/javafx/textformatter1"
    btn_label: "Read More"
    btn_class: "btn--primary"
  - image_path: /assets/images/LittleBrain.jpg
    alt: "placeholder image 2"
    title: "Part 2"
    excerpt: "This second article dives deep into details of the `Filter` and how to use it to create a sophisticated `TextFormatter` that will completely transform the way that a `TextField` operates."
    url: "/javafx/textformatter2"
    btn_label: "Read More"
    btn_class: "btn--primary"
  - image_path: /assets/images/LittleBrain.jpg
    title: "Part 3"
    excerpt: "The last article in this series carries on from Part 2 to add some additional handling to the `TextFormatter`.<br><br>**Coming Soon**"
    btn_label: "Read More"
    btn_class: "btn--primary"
---
# TextFormatter

`TextFormatter` is the coolest JavaFX feature that you''ve probably never heard of.  With `TextFormatter` you can turn any `TextField`  or `TextArea` into a customized data entry field that will accept only a specific kind of input and bound directly to any type of `Property` in your Data Model.  You can create `TextFields` for Numbers, Dates, Phone Numbers or Postal Codes.  Really, anything you can think of.

{% include notice type="info" content = "As with almost everything else in JavaFX: Here we are given some really powerful and well designed low-level tools and we're left to figure out how to do something useful with them. " %}

There's no standard implementations of `TextFormatter` in the library to handle even typical situations like integer or decimal number input, and nothing to use a starting point to build your own `TextFormatter`.  This can be daunting.

This section should help you.  

## How TextFormatter Works

`TextFormatter` is a component that you install onto a `TextField` which then provides two interfaces.  One to your Data Model and one for the user interaction.

### The Converter

The `Converter` is a bi-directional transform that converts the `String` data in your `TextField` to an external format or type.  This means that you can have the types and the format of the Properties in your Data Model match their purpose, without having to worry about how they will be represented in the GUI.  

You could, for instance provide an `IntegerProperty` in your Model, and have the `Converter` translate it to the `String` required by `TextField`.  

Additionally, you could have a `StringProperty` in your Model, but its format is different from the best representation in the GUI.  For instance you could have phone numbers stored in your model with just the digits, something like "9995551212", but have the TextField display "(999)555-1212".  Converter will handle that for you, and you can directly bind the Model Property to the Value Property in the `TextFormatter`.

### The Filter

The `Filter` is the interface that sits between the `TextField` itself and all of the user interactions.  This includes keystrokes and mouse actions, including selecting text inside the TextField.

No more trying to capture `KeyEvents` and interpreting them!  The Filter does it for you.

The `Filter` is called "Filter" because each user action is packaged up as a "Change" and passed through the `Filter` before it is applied to the `TextField`.  The `Change` object contains virtually all of the information that you need to evaluate the impact of the change on the `TextField`.  The `Filter` can either let the change go through as is, halt the change, or let a modified version of the change pass on to the `TextField`.

### Putting it Together

The basic process is straight forward.  Create a `Converter` that will handle your data type and then a `Filter` that will ensure that the `TextField` will behave the way you want.  Then you pass them to the constructor of a `TextFormatter` that install onto a `TextField` or a `TextArea` using its `setTextFormatter()` method.

## The Articles

This is a big subject, and cannot be adequately handled in a single article.  So it's been divided into three:

{% include feature_row %}
