---
layout: single
title: TableView and ListView
toc: false
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
permalink: /javafx/elements/tableview_listview
skip_link: true

feature_row_table_basics:
  - image_path: /assets/elements/TableView2.png
    alt: "Table Basics"
    title: "TableView Basics"
    excerpt: 'Everything you need to know to get started using TableView in your layouts.'
    url: "/javafx/elements/tableview-basics"
    btn_label: "Read The Article"
    btn_class: "btn--primary"

feature_row_cell_data:
  - image_path: /assets/elements/TableView9.png
    alt: "Cell Data"
    title: "TableCell and TableColumn Data"
    excerpt: 'Learn how data gets from your ObservableList into your TableCells and how to cope with situations where the data gets a bit more complicated.'
    url: "/javafx/elements/tableview-data"
    btn_label: "Read the Article"
    btn_class: "btn--primary"

feature_row_listview_basics:
  - image_path: /assets/elements/ListView2.png
    alt: "ListView Basics"
    title: "Understanding ListView"
    excerpt: 'Learn the basics about ListView and how to customize the display of simple data.'
    url: "/javafx/elements/listview-basics"
    btn_label: "Read the Article"
    btn_class: "btn--primary"  

feature_row_listview_layouts:
  - image_path: /assets/elements/ListView5.png
    alt: "ListView Cell Layouts"
    title: "ListView Cell Layouts"
    excerpt: 'Learn how to customize ListView cell layouts to handle complex data structures and interactive data presentation'
    url: "/javafx/elements/listview-layouts"
    btn_label: "Read the Article"
    btn_class: "btn--primary"  
---

# TableView

We all know what tables look like.  Rows and columns of data that feel a little bit like spreadsheets once you've got enough rows and columns to need scrollbars.  `TableView` is the JavaFX layout element to create tables in your layouts.

For some reason, beginner programmers love tables, and they rush to start using them as soon as possible.  Unfortunately, the official documentation for JavaFX and most of the information you'll find on web about `TableViews` is lacking in insight.  The articles here take a different look at `TableView`, and try to explain how things really work "under the hood" so that you can make informed decisions about how to code up your own `TableViews`

## The Articles

{% include feature_row id="feature_row_table_basics" type="left" %}

{% include feature_row id="feature_row_cell_data" type="left" %}

# ListView

`ListView` is one of the most common elements that you'll find in JavaFX, yet it's under-used by most programmers.  It's possible to look at a `ListView` as just a `TableView` with a single column, but there are some fundamental differences that make it much easier to use `ListView` to present complicated layouts that display complex data structures.

## The Articles

{% include feature_row id="feature_row_listview_basics" type="left" %}

{% include feature_row id="feature_row_listview_layouts" type="left" %}
