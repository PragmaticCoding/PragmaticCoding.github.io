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
feature_row_hangman:
  - image_path: /assets/posts/Hangman.png
    alt: "Hangman"
    title: "Hangman"
    excerpt: 'The classic game "Hangman" written in Reactive JavaFX.'
    url: "/javafx/hangman"
    btn_label: "Read More"
    btn_class: "btn--primary"
feature_row_minesweeper:
  - image_path: /assets/posts/MineSweeper1.png
    alt: "MineSweeper"
    title: "MineSweeper"
    excerpt: 'The classic game "MineSweeper" written in Reactive JavaFX using a multi-layered MVC design.'
    url: "/javafx/minesweeper"
    btn_label: "Read More"
    btn_class: "btn--primary"
feature_row_wordle:
  - image_path: /assets/posts/WordleFX1.png
    alt: "WordleFX"
    title: "WordleFX"
    excerpt: 'The popular game, "Wordle" implemented as a desktop application, written in Kotlin using Reactive JavaFX'
    url: "/javafx/wordle"
    btn_label: "Read More"
    btn_class: "btn--primary"

feature_row_weather:
  - image_path: /assets/posts/Weather.png
    alt: "WeatherFX"
    title: "WeatherFX"
    excerpt: 'A sample application with an MVC Reactive JavaFX framework connected to an external API through a domain service and and Interactor. <br>This application is intended to demonstrate how you can write a JavaFX application that "does something".'
    url: "/javafx/weather"
    btn_label: "Read More"
    btn_class: "btn--primary"    
---

{% include feature_row id="intro" type="center" %}

{% include feature_row id="feature_row_weather" type="left" %}

{% include feature_row id="feature_row_hangman" type="left" %}

{% include feature_row id="feature_row_wordle" type="left" %}

{% include feature_row id="feature_row_minesweeper" type="left" %}
