---
layout: single
title: Custom JavaFX Components
permalink: /javafx/elements/custom_components_page
toc: false
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
skip_link: true
feature_row_intro:
  - image_path: /assets/elements/CustomClass1.png
    alt: "Getting Started with Custom Components"
    title: "Getting Started with Custom Components"
    excerpt: 'You can get started with custom components by simply building a library of factory/builder methods that create styled standard components or simple sub-layouts that you use all the time.  From there you can move on to create custom classes that you can use just like the standard JavaFX Nodes in your layouts.'
    url: "/javafx/elements/custom_controls"
    btn_label: "Read The Article"
    btn_class: "btn--primary"

feature_row_customizing:
  - image_path: /assets/elements/CustomClass5.png
    alt: "Customizing"
    title: "Crafting a Custom Component Class"
    excerpt: "Now that we have a custom component, let's add the styling elements to make it work just like a standard JavaFX Node.  By the end of this article you will understand how to create styleable properties that your client code can use in a style sheet to customize your component."
    url: "/javafx/elements/customizing_custom_controls"
    btn_label: "Read the Article"
    btn_class: "btn--primary"

feature_row_triple_switch:
  - image_path: /assets/elements/ThreeWayToggle.png
    alt: "TripleSwitch"
    title: "Custom Component Example: Triple Switch"
    excerpt: "Here's an example of how to create a component that's simply missing from JavaFX, a three-way Toggle switch.  In this article we'll use the technique of extending the Region class to create a custom component that works just like any other JavaFX Node."
    url: "/javafx/elements/tripletoggle"
    btn_label: "Read the Articles"
    btn_class: "btn--primary"    
---

JavaFX has a large library of standard components that you can use to create your layouts.  

There's nothing magic about these components, and if you look at the source code you'll find that they're just Java classes that use standard Java and JavaFX techniques.  This means that you can create your own!

At it's simplest, a custom component can just be a standard JavaFX `Control` you style a certain way all the time.  Rather than repeat yourself in every layout, you can add static method to a library class somewhere that instantiates, styles and returns that control.  You might have sub-layout patterns that you use consistently, and those too can be put into your library class.  

Finally, you might decide that you want to create a custom component class of your own, that you can drop into any layout and style and configure in the same way that you would any of the standard JavaFX `Nodes`.  

This is not hard to do.  If you can write and configure a JavaFX layout, then you can create your own component class.  These articles here will show you how to do it!



## The Articles:

{% include feature_row id="feature_row_intro" type="left" %}

{% include feature_row id="feature_row_customizing" type="left" %}

{% include feature_row id="feature_row_triple_switch" type="left" %}
