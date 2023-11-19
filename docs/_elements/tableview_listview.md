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
  
---

# TableView

We all know what tables look like.  Rows and columns of data that feel a little bit like spreadsheets once you've got enough rows and columns to need scrollbars.  `TableView` is the JavaFX layout element to create tables in your layouts.

For some reason, beginner programmers love tables, and they rush start using them as soon as possible.  Unfortunately, the official documentation for JavaFX and most of the information you'll find on web about `TableViews` is lacking in insight.  The articles here take a different look at `TableView`, and try to explain how things really work "under the hood" so that you can make informed decisions about how to code up your own `TableViews`

## The Articles

{% include feature_row id="feature_row_table_basics" type="left" %}

{% include feature_row id="feature_row_cell_data" type="left" %}
