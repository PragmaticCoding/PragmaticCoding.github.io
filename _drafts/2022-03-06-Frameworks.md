---
title:  "Unravelling MVC, MVP and MVVM"
date:   2022-02-06 12:00:00 -0500
categories: javafx
logo: /assets/logos/LittleBrain.jpg
excerpt: Taking a look at the three most common design patterns for building systems with user interfaces.  How are they different?  Which one is best?  Is there anything better?
---

# Introduction

Model-View-Controller (MVC), Model-View-ViewModel (MVVM) and Model-View-Presenter (MVP) are three of the most common design patterns that you are likely to encounter when working with systems that have a User Interface (UI).  

## What's Their Purpose

These are *design patterns*.  They are standard ways of architecting things to achieve common ends.  The advantage to using a design pattern is that they are generally well known and well thought-out, so they are familiar and easy to build and to understand when you encounter them in the real world.

In this particular case, these are different approaches to solve the problem of separating the presentation of data to the user from the actual data and the back-end logic.  In this way, the UI becomes much less dependent on the back-end structure (and visa versa) so that changes to either the UI or the back-end don't require huge changes to other end.

# Understanding the Terminology

A small amount of time spent searching the web will show you that there's many different interpretations of what design patterns really mean.  Even if you go back to some of the original articles where these patterns were first introduced, there's a lot of ambiguity and room for interpretation.  

So let's look at the most confusing concept first:

## The "Model"

There's an idea that data should be bundled up with code that is related to it, so that the results is a response object rather than just a bunch of data in fields.   This is a common concept in OOP system design, and you've undoubtedly seen it many, many times.  Even if it's just getters and setters to isolate the internal implementation of the data from it's outward presentation.

That's the concept behind "Model" in all of these design patterns.  It's not just the data, but it's the data handling as well.  This means the business logic, and calls to external API's and a persistence layer.  All of it.

Often it's referred to as the "Domain Data"

## Domain Data

This is where we get into more trouble.  For me "domain data" has always meant data related to the context of the application.  For instance, if we're working with an Sales application, then domain data is things like orders, invoices, customers and transactions.

But quite often when looking at the descriptions of these design patterns, it's clear that the meaning of "domain data" can be very contextual.  It's possible that the GUI construct becomes the "domain", and any data related to it, both screen related data and back-end application data.

This seems to be the most common use of "Domain Data" when talking about these design patterns.

# How Do the Patterns Work?

The first thing to understand is that all of these patterns have the same goal, to create independence between the business logic and the presentation that the user interacts with.  They all have a great deal of similarity, but they have some profound differences in the details.

Let's look at each of these patterns and try to figure out the best understanding of what they mean.  We'll start with MVP since it's probably the least useful with modern, reactive UI design:

## Model-View-Presenter

### Overview

This pattern is probably the closest fit for FXML screens.  The FXML file and the "root" Node that is created by the FXMLLoader, would be the View, and the FXML Controller would take the role of Presenter.  The FXML structure pretty much ends there, and doesn't speak to the role of "Model".  Many programmers seem to have no problem putting SQL queries and file handling right in the FXML Controller, along with loads of business logic, so the role of Model is often merged into the FXML Controller.

### The Model

As previously discussed, the Model has all of the data, and all of the business logic contained within it.  

### The View

In MVP the View is supposed to be an entirely passive entity.  It contains virtually no decision-making logic of it's own, and it's entire purpose is to put data on the screen, and to receive input from the user.  Once it gets input from the user, its only role is to pass it on to the Presenter, which will decide how to handle it.

### The Presenter




## Model-View-ViewModel

### Overview

At first glance MVVM appears to be the best match for modern, reactive UI's as it specifically talks about "binding" to connect the View to the ViewModel.  

### The ViewModel

The ViewModel contains the data in a structure appropriate to bind it to various components of the View.  It also contains the functionality to prepare the data in that format.

### The Model

The Model in this pattern *potentially* leans more towards dealing with application domain data, as the ViewModel handles some amount of the transformation of this data into presentation data.

### The View

## Model-View-Controller

### The Model

### The View

### The Controller
