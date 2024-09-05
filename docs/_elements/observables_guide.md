---
layout: single
title: JavaFX Nodes
toc: false
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
permalink: /javafx/elements/observables_guide
skip_link: true


feature_row_part1:
  - image_path: /assets/elements/Properties.png
    alt: "Generic Observables"
    title: "The Generic Observable Types"
    excerpt: "This is the starting place for understanding the Observables.  In this article we look at the Observables that wrap generic Object types as their value.  This is also the article that looks the deepest into the classes and interfaces that are repeated in the later articles."
    url: "/javafx/elements/observable-classes-generics"
    btn_label: "Read the Article"
    btn_class: "btn--primary"    
feature_row_part2:
  - image_path: /assets/elements/IntegerProperties.png
    alt: "Typed Observables"
    title: "The Typed Observables"
    excerpt: "There are a set of Observables designed to wrap specific value types, like String, Boolean and Integer.  In this article we look at how these typed Observables differ from the generic Observables and how you should use them."
    url: "/javafx/elements/observable-classes-typed"
    btn_label: "Read the Article"
    btn_class: "btn--primary"    
feature_row_part3:
  - image_path: /assets/elements/ListProperties.png
    alt: "List Observables"
    title: "The List Observables"
    excerpt: "In this article we look at the last type of Observables, the mysterious List Observables that both wrap ObservableLists and act like ObservableLists themselves.  Why would you use these?  What can you do with them?  The answers are much more interesting than you might think.  Don't skip this article."
    url: "/javafx/elements/observable-classes-lists"
    btn_label: "Read the Article"
    btn_class: "btn--primary"    

---

# The Observable Types

Observable types are the basic building blocks of creating Reactive JavaFX applications, but they can be a bit confusing.  There are so many of them.

What are they all used for?  What makes them different from each other?  Which ones should you use, and where?

At it's simplest, `Observables`, `Bindings` and `Properties` are all wrappers for a value that allow for the "Observer Pattern" to be implemented.  All of these classes and interfaces can be "observed" and many of them can act as "observers" themselves.  This allows you to connect your data together so that a change to a `Observable` somewhere in a data model or `Node` can trigger actions and changes all over your application.

The articles on this page comprise a complete look at all of the various interfaces and classes that make up the `Observables` library in JavaFX in a way that is easy to understand.  This is a huge subject, so it's divided into three parts, each one building on the previous and explaining how the various pieces can be best used.

These articles are aimed at readers that already have a basic familiarity with `Observables` and some experience using them.  If you are one of those programmers they will probably answer a lot of questions that you didn't even know you had.  Things that bugged you subconsciously, or that you just skipped over as "unknowable".

# The Articles:

{% include feature_row id="feature_row_part1" type="left" %}

{% include feature_row id="feature_row_part2" type="left" %}

{% include feature_row id="feature_row_part3" type="left" %}
