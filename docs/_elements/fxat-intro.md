---
layout: single
title: Understanding the FXAT
toc: false
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
permalink: /javafx/elements/fxat-intro
skip_link: true
feature_row_intro:
  - image_path: /assets/elements/BackgroundProcess_Square.png
    alt: "Intro"
    title: "Introduction to the FXAT"
    excerpt: 'The basic beginners guide to the FXAT.  What you really need to know to get started, including how to get off the FXAT to run long jobs using Task'
    url: "/javafx/elements/fxat"
    btn_label: "Read The Article"
    btn_class: "btn--primary"

feature_row_task_progress:
  - image_path: /assets/elements/TaskProgress1.png
    alt: "TaskProgress"
    title: "Tracking the Progress of Tasks"
    excerpt: 'A deeper look into Task to understand how to track progress in your GUI while a Task is running'
    url: "/javafx/elements/task-progress"
    btn_label: "Read the Article"
    btn_class: "btn--primary"

feature_row_imageview:
  - image_path: /assets/elements/ImageViewDemo1.png
    alt: "ImageView"
    title: "All About Image and ImageView"
    excerpt: "Image and ImageView are the two classes you'll need to know in order to be able to put images into your layouts."
    url: "/javafx/elements/images"
    btn_label: "Read the Articles"
    btn_class: "btn--primary"    

feature_row_layouts:
  - image_path: /assets/elements/VBoxHBox.png
    alt: "Layouts"
    title: "Introduction to Layout Classes"
    excerpt: "A basic guide to the layout classes in JavaFX and how to use them effectively"
    url: "/javafx/elements/layout_classes"
    btn_label: "Read the Article"
    btn_class: "btn--primary"

feature_row_tables:
  - image_path: /assets/elements/TableView2.png
    alt: "Tables and Lists"
    title: "Introduction to TableView and ListView"
    excerpt: "A set of articles about how to use the JavaFX tools to create lists and tables in your layouts."
    url: "/javafx/elements/tableview_listview"
    btn_label: "Read the Articles"
    btn_class: "btn--primary"       
---

# What is the FXAT?

When a library is "Thread-Safe" it means that it has been designed such that multiple processes, or multiple threads, can access the same objects at the same time (or at least seem to access them at the same time).  This introduces a massive amount of complexity into the library, as there need to be controls to ensure that one thread doesn't update a field while another thread is depending on it still having the same value that it did 1ms ago.  

The JavaFX library is NOT thread-safe, and this greatly simplifies its structure and makes it easier to work with.  However, this means that the objects used on the SceneGraph need to be accessed by only one single thread.  And that thread is the "FX Application Thread" or "FXAT".  

The operation that gets your application up and running and on the screen through the `Application.start()` method also starts up the FXAT and gets your code running on it. Knowing how to get your code to run on or off the FXAT is an important part of JavaFX programming.  The articles on this page will help you get started.


# The Articles:

{% include feature_row id="feature_row_intro" type="left" %}

{% include feature_row id="feature_row_task_progress" type="left" %}
