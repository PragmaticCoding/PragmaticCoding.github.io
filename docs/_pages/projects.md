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

intro:
  - excerpt: 'These are links to some of the projects I''ve worked on lately and published on GitHub.  If you find any of the programming techniques useful or interesting, feel free to fork these projects.'
feature_row:
  - image_path: /docs/assets/images/Hangman.png
    alt: "placeholder image 1"
    title: "Placeholder 1"
    excerpt: "This is some sample content that goes here with **Markdown** formatting."
  - image_path: /docs/assets/images/HangedMan.png
    alt: "placeholder image 2"
    title: "Placeholder 2"
    excerpt: "This is some sample content that goes here with **Markdown** formatting."
    url: "#test-link"
    btn_label: "Read More"
    btn_class: "btn--primary"
  - image_path: /assets/logos/JavaFXLogo.png
    title: "Placeholder 3"
    excerpt: "This is some sample content that goes here with **Markdown** formatting."
feature_row2:
  - image_path: /assets/images/unsplash-gallery-image-2-th.jpg
    alt: "placeholder image 2"
    title: "Placeholder Image Left Aligned"
    excerpt: 'This is some sample content that goes here with **Markdown** formatting. Left aligned with `type="left"`'
    url: "#test-link"
    btn_label: "Read More"
    btn_class: "btn--primary"
feature_row3:
  - image_path: /docs/assets/images/Hangman.png
    alt: "Hangman"
    title: "Hangman"
    excerpt: 'The classic game "Hangman" written in Reactive JavaFX.'
    url: "/javafx/hangman"
    btn_label: "Read More"
    btn_class: "btn--primary"
feature_row4:
  - image_path: /docs/assets/images/MineSweeper1.png
    alt: "MineSweeper"
    title: "MineSweeper"
    excerpt: 'The classic game "MineSweeper" written in Reactive JavaFX using a multi-layered MVC design.'
    url: "/javafx/minesweeper"
    btn_label: "Read More"
    btn_class: "btn--primary"
---

{% include feature_row id="intro" type="center" %}

{% include feature_row id="feature_row3" type="left" %}

{% include feature_row id="feature_row4" type="left" %}
