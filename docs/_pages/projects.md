---
layout: single
title: Projects
toc: false
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4
sidebar:
    nav: "master"
permalink: /pages/projects/
brain: /assets/logos/brain.png
intro:
  - excerpt: 'These are links to some of the projects I''ve worked on lately and published on GitHub.  If you find any of the programming techniques useful or interesting, feel free to fork these projects.'
feature_row1:
  - image_path: /assets/posts/Hangman.png
    alt: "Hangman"
    title: "Hangman"
    excerpt: 'The classic game "Hangman" written in Reactive JavaFX.'
    url: "/javafx/hangman"
    btn_label: "Read More"
    btn_class: "btn--primary"
feature_row2:
  - image_path: /assets/posts/MineSweeper1.png
    alt: "MineSweeper"
    title: "MineSweeper"
    excerpt: 'The classic game "MineSweeper" written in Reactive JavaFX using a multi-layered MVC design.'
    url: "/javafx/minesweeper"
    btn_label: "Read More"
    btn_class: "btn--primary"
---

{% include feature_row id="intro" type="center" %}

{% include feature_row id="feature_row1" type="left" %}

{% include feature_row id="feature_row2" type="left" %}
