---
layout: single
title: Stylesheets and PseudoClasses
toc: false
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
permalink: /javafx/elements/stylesheets_pseudoclasses
skip_link: true

feature_row_css:
  - image_path: /assets/elements/StandardColourShot.png
    alt: "CSS"
    title: "Getting Started With CSS"
    excerpt: 'How to get started styling your JavaFX applications using cascading style sheets.'
    url: "/javafx/elements/stylesheets"
    btn_label: "Read The Article"
    btn_class: "btn--primary"

feature_row_pseudoclass_intro:
  - image_path: /assets/elements/PseudoClass3.png
    alt: "PseudoClasses"
    title: "Getting to Know PseudoClasses"
    excerpt: "When you need to have the styling of a screen element change according to the status of something in your application, PseudoClasses are the way to go.  Here's what you need to know to get started with PseudoClasses"
    url: "/javafx/elements/pseudo_classes"
    btn_label: "Read The Article"
    btn_class: "btn--primary"

feature_row_enum_pseudoclass:
  - image_path: /assets/elements/EnumPseudo2.png
    alt: "Enum PseudoClass"
    title: "Non-Binary PseudoClasses"
    excerpt: 'PseudoClasses are usually binary on/off what if you have a variety of different values that want to use to control temporary styling on your GUI elements.  Non-Binary PseudoClass Properties can solve the problem.'
    url: "/javafx/nonbinary-pseudoclass"
    btn_label: "Read the Article"
    btn_class: "btn--primary"
---

# JavaFX nodes

There are large amount of classes in JavaFX arranged in a hierarchy starting with a top level class called, `Node`.  There are a number of layout classes that all work differently, such as `BorderPane`, `ScrollPane`, `HBox`, `VBox`, `GridPane` and `StackPane` which can all be embedded inside of each other to create exactly the layout you want.

There are classes to display images, text or text and images together.  `ListView`, `TreeView` and `TableView` allow collections of data to be displayed on screen (and edited) in highly customizable ways.

For user input, you can choose from `TextField`, `ComboBox`, `ChoiceBox`, `Spinner`, `Checkbox`, `RadioButton`, `Buttons` and `ToggleButtons`.

All of these Nodes have a large number of methods and properties for controlling their presentation and how they act.  Learning how to use them and configure them can be a daunting task.  Luckily, most of the tutorials on the web seem to focus on this aspect of JavaFX, so help is easy to find.

# The Articles:

{% include feature_row id="feature_row_css" type="left" %}

{% include feature_row id="feature_row_pseudoclass_intro" type="left" %}

{% include feature_row id="feature_row_enum_pseudoclass" type="left" %}
