---
layout: single
title: Observable Lists
toc: false
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
permalink: /javafx/elements/observable_lists
skip_link: true


feature_row_beginners:
  - image_path: /assets/logos/LittleBrain.jpg
    alt: "Beginners Guide"
    title: "The Beginners Guide to ObservableLists"
    excerpt: "This is the starting place for understanding the ObservableLists.  In this article we cover all the basics that you need to master to be able to use ObservableLists effectively."
    url: "/javafx/elements/observable-lists-basics"
    btn_label: "Read the Article"
    btn_class: "btn--primary"    
feature_row_extractors:
  - image_path: /assets/logos/LittleBrain.jpg
    alt: "Extractors"
    title: "ObservableList Extractors"
    excerpt: "While ObservableLists will tell you when items have been added or removed, what if your List items are composed of Properties that can change?  Can you get an ObservableList to fire a Listener when one of the component fields changes?  Sure you can.  This article will show you how."
    url: "/javafx/elements/extractors"
    btn_label: "Read the Article"
    btn_class: "btn--primary"    
feature_row_list_properties:
  - image_path: /assets/elements/ListProperties.png
    alt: "List Observables"
    title: "The List Observables"
    excerpt: 'In this article we look at ListProperty, the mysterious ObjectProperty<ObservableList> that both wraps an ObservableLists and acts like an ObservableList itself.  Why would you use these?  What can you do with them?  The answers are much more interesting than you might think.  This is an advanced article, and is part of my "Complete Guide to the Observable Types"'
    url: "/javafx/elements/observable-classes-lists"
    btn_label: "Read the Article"
    btn_class: "btn--primary"    

---

# Getting to Know ObservableLists

At their simplest, `ObservableLists` are just `Lists` that can register `Listeners` fire whenever elements are added, removed or moved in the `List`.

But, of course, there's much more to it than that.  

To begin with, it's very hard to avoid `ObservableLists` since they are critical to `TableView` and `ListView`.  And, since they are used in `ListView` they are also used in `ComboBox` and `ChoiceBox`.  Pretty much anywhere a `List` is used in JavaFX, it's implemented as an `ObservableList`.  When you call `HBox.getChildren()`, you'll get an `ObservableList` back.

So it's very important to know how to deal with `ObservableList`, and there's also a catalogue of cool utilities and speciality classes that make it easy to do some fairly sophisticated things with `ObservableLists`.  To get the most out of JavaFX, it's important to know about these, too.

# The Articles:

{% include feature_row id="feature_row_beginners" type="left" %}

{% include feature_row id="feature_row_extractors" type="left" %}

{% include feature_row id="feature_row_list_properties" type="left" %}
