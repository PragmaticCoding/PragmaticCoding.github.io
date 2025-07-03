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

feature_row_list_progress:
  - image_path: /assets/elements/TaskProgress10.png
    alt: "ImageView"
    title: "Tracking Task List Building Progress"
    excerpt: "Sometimes you need a Task to return not just a single value, but a List of values.  This article explores how to monitor the progress of your Task as it accumulates data into a List, and to see that List as it grows from your GUI."
    url: "/javafx/elements/task-list-progress"
    btn_label: "Read the Article"
    btn_class: "btn--primary"    
---

# What is the FXAT?

When a library is "Thread-Safe" it means that it has been designed such that multiple processes, or multiple threads, can access the same objects at the same time (or at least seem to access them at the same time).  This introduces a massive amount of complexity into the library, as there need to be controls to ensure that one thread doesn't update a field while another thread is depending on it still having the same value that it did 1ms ago.  

The JavaFX library is NOT thread-safe, and this greatly simplifies its structure and makes it easier to work with.  However, this means that the objects used on the SceneGraph need to be accessed by only one single thread.  And that thread is the "FX Application Thread" or "FXAT".  

Everything that deals with the GUI has to run on the FXAT.  This includes any code that you write that updates `Properties` of `Nodes` on the screen, as well as all of the internal JavaFX code that runs the screen updates, moves the cursor, responds to mouse clicks, window resizing and anything else you can think of.  All of these actions need to run on that single FX Application Thread.  

But if you monopolize the FXAT with your application code, then your GUI will start to stutter and become unresponsive.  This can happen because you've done one of two things:

1. Run long, complicated operations that take more than a few milliseconds on the FXAT.
1. Run operations that block (like accessing a web service) on the FXAT.

You need to do these things on a "background" thread.  

The operation that gets your application up and running and on the screen through the `Application.start()` method also starts up the FXAT and gets your code running on it. Knowing how to get your code to run on or off the FXAT is an important part of JavaFX programming.  The articles on this page will help you get started.


# The Articles:

{% include feature_row id="feature_row_intro" type="left" %}

{% include feature_row id="feature_row_task_progress" type="left" %}

{% include feature_row id="feature_row_list_progress" type="left" %}
