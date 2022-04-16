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
permalink: /javafx/elements/images
skip_link: true


feature_row_imageview:
  - image_path: /assets/elements/ImageViewDemo1.png
    alt: "ImageView"
    title: "All About Image and ImageView"
    excerpt: "Image and ImageView are the two classes you'll need to know in order to be able to put images into your layouts.  This article will give you all the information you need to get started with these classes."
    url: "/javafx/elements/imageview"
    btn_label: "Read the Article"
    btn_class: "btn--primary"    
feature_row_sprites:
  - image_path: /assets/elements/SpriteScreenShot3.png
    alt: "ImageView"
    title: "Image Animation with Sprites and Scrolling Backgrounds"
    excerpt: "How to use Sprites in ImageView to create animation.  How to create infinite scrolling backgrounds.  Even better, this article has Trolls!!"
    url: "/javafx/elements/sprites"
    btn_label: "Read the Article"
    btn_class: "btn--primary"    
---

# Image and ImageView

`ImageView` is the basic JavaFX `Node` to display images in your GUI.  `Image` is the class used to hold the data contained within an `ImageView`.  The relationship between the two is very similar to the relationship between `Label` and `String`.

Mastering `ImageView` is fairly simple and allows you to add a lot of extra flair to your layouts.  From putting images on `Buttons`, to adding logos and image backgrounds, `ImageView` is the essential class to know.

# The Nodes:

{% include feature_row id="feature_row_imageview" type="left" %}

{% include feature_row id="feature_row_sprites" type="left" %}
