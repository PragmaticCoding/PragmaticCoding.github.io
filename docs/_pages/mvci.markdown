---
layout: single
title: Model-View-Controller-Interactor
toc: true
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
permalink: /javafx/mvci/
skip_link: true
Diagram: /assets/posts/MVCI.png
---
# Model-View-Controller-Interactor

For years I struggled to find a way to fit Reactive JavaFX into one of the commonly use frameworks: Model-View-Presenter, Model-View-Controller or Model-View-ViewModel.  But none of them really seemed to work.

First, they're all commonly misunderstood.  If you look at different sources, you'll find different definitions for each.  There's a lot of copypasta, and then explanations and examples that seem to contradict the definitions provided.  Problems with each of these frameworks are often ignored, and sample code breaks the rules.

One of the reasons for these contradictions is that none of these frameworks is actually a good fit for Reactive JavaFX design!

As a developer and a manager with a mandate to produce working JavaFX applications to support enterprise business I had to find a better framework.  The result is Model-View-Controller-Interactor.

![Diagram]({{page.Diagram}})

It's a bit like the other frameworks.  It has the same goals as the other frameworks.  But it's not just an adaptation of  those other frameworks.  

And it works.

It makes designing Reactive JavaFX applications much more straight-forward.  It handles complexity by controlling coupling, and it puts everything in the right place.  


# The Introduction

Want to know more?  Then start here.  This is an article that describes everything you need to know about why you should use MVCI and how to go about using it:

[Read the Introduction](/javafx/Mvci-Introduction){: .btn .btn--info}

# It Scales!

Model-View-Controller-Interactor allows you to construct a framework that defines a single function or screen.  But what if you have a much bigger application that has many, many functions and many different screens?  What if you have common functions that repeat over several screens?  

With MVCI, it's really easy to connect multiple frameworks to each other in a number of different ways.  Here's an article that shows how to put together most of the scenarios you're likely to encounter:

[Building Multi-Screen Applications with MVCI](/javafx/multimvci){: .btn .btn--info}

# What's Wrong with the Other Frameworks?

These other frameworks have been around for years.  They were dreamed up by really smart people who knew what they were doing.  So why do we need another framework?

The truth is that those frameworks were invented decades ago.  At least some of them were invented long before Java was around, and all of them were created before JavaFX.  For the most part, they are highly theoretical and described in abstract terms and don't account for the practicalities of modern computer languages and development tools.  In particular, they don't dovetail nicely with the concepts around Reactive programming.

Another side-effect of the abstract nature of their definitions is that they are very difficult to understand and, therefore, are commonly misunderstood.  If you search the web for guidance about any of these frameworks, you'll find no shortage of information, much of it incomplete, inaccurate and contradictory.  The result is that developers are left to decide for themselves how to approach implementing a framework and this leads to complications and issues down the road.  

After years, I spent a lot of time researching these frameworks to try to get to the truth.  How can you use them in a Reactive programming environment, and what fine-tuning do they need?  Is something new needed?

If you're interested in my conclusions about this, you can read this article here:

[Unravelling MVC, MVP and MVVM](/javafx/Frameworks){: .btn .btn--info}


# FXML is Not MVC

There's a widely held fallacy out there that using FXML automatically means you're using MVC.  I large part of this confusion has to do with the fact that the authors of JavaFX chose to call the code part of FXML the "Controller".  However, the FXML Controller has a very different role from that of the MVC Controller, and it's disastrous to treat your FXML Controller like an MVC Controller.  The result is almost always monolithic, single-class applications with a spaghetti of business logic and View code in one place.  

If you're curious about this, or if you want to learn how to integrate FXML into MVC (or MVCI) the *right* way then read this article here:

[FXML is NOT MVC](/javafx/fxml_isnt_mvc){: .btn .btn--info} 
