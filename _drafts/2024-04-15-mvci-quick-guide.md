---
title:  "JavaFX: Quick Guide to MVCI"
date:   2024-03-09 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/mvci-quick
ScreenSnap1: /assets/posts/StarterFX.png
ScreenSnap2: /assets/posts/StarterFX.png
ScreenSnap3: /assets/posts/StarterFX.png

excerpt: Looking for a framework that works with JavaFX as a Reactive platform?  Here's how to get started with Model-View-Controller-Interactor (MVCI) without a lot of theory.
---

# Introduction - Why Use a Framework?

If you are looking to write a GUI application it is important to organize your application so that it will be easy to maintain, enhance and expand over time.  

# What's Wrong With the Others?

*  Model-View-Presenter (MVP)<br>The problem with MVP is that the View is stripped down to nothing more than layout and styling, leaving all other aspects of the View to be handled by the Presenter.  This requires a level of coupling between the View and the Presenter that is so tight that they might as well be considered to be a single component.  The only aspect of the View that remains uncoupled is the actual layout, which is arguably the least import coupling concern.  I'm not sure that anybody uses MVP any more.

*  Model-View-ViewModel (MVVM)<br>MVVM looks like a much better fit for Reactive JavaFX because it specifically allows for `Binding` between the Presentation Model and the View.  However, the ViewModel isn't allowed to see the Domain Objects and the Model isn't allowed to see Presentation Model and the result is that the ViewModel and the Model get bogged down passing information between them.  The coupling is loose, but it becomes a problem itself because it becomes complicated to work around it.

*  Model-View-Controller (MVC)<br>MVC solves many of the issues with MVVM because the Presentation Model is *part* of the Model.  However, MVC specifically exposes the Model (and, therefore, the Presentation Model) to the View in a "read-only" mode.  This means that `Properties` in the Model cannot be bound to the `Properties` of the `Nodes` in the View and updates must be passed through the Controller to the Model.

# What is Model-View-Controller-Interactor?

MVCI solves the problems of these other frameworks by taking MVC and splitting the Model into two parts: the Presentation Model (just called the "Model" in this framework) and the "Interactor".  This new Model is simple a POJO composed of JavaFX `Observable` objects and lists, and the other three components all have read and write access to it.  

The result is something that is instantly simple and easy to integrate with Reactive JavaFX.  `Properties` of the `Nodes` in the View can be bound in either direction to the elements of the Model.  This means that the Model is truly a data representation of the "state" of the GUI at all times.  At the other end of the framework, all of the business logic is contained and isolated within the Interactor, which also is the only component that can access external services, applications and databases.

This fulfils the purpose of this kind framework.  Business logic, application connective tissue and GUI design are all isolated from each other and very loosely coupled.  The main source of coupling is the Model, which serves as a "contract" to which the other components need to adhere.  As long as the structure of the Model is preserved, any implementation of business logic or GUI design can be employed with affecting any other component of the framework.

# The Components of MVCI

Let's take a look at the four components of MVCI and see what they are how they work together.

## The Model

The Model is just a bunch of data.  But it's data stored in JavaFX `Observable` objects (typically `Properties`), and `ObservableLists`.  

## The Controller

The Controller is the connective tissue of the framework.  It instantiates all the other components, and it handles anything involving threading (which becomes important when you need to deal with long or blocking processes off the FXAT).  It also contains any links back to other MVCI frameworks in your application, usually through shared Model elements.  Typically, you will initiate an MVCI framework by instantiating the Controller, and it will bootstrap itself through it's constructor code.

## The Interactor

Here's where the business logic resides, and the calls to your external services, databases and applications.  It's also the only place that you'll find "Domain Objects".  Domain Objects are the classes that hold the data that are returned from (and are sent to) your "Services" and they are completely foreign to your View and your Controller.  

## The View and the ViewBuilder

The View is a fully functioning JavaFX `Node` subclass that can be added to the SceneGraph.  It handles all of the display, and any direct interaction with the user.

There are two approaches to creating a View.  One way is to create a custom JavaFX component by extending one of the JavaFX classes like `Region`.  The general rule in JavaFX, however, is to "extend only when adding new functionality".  In this sense, "adding new functionality" is equivalent to adding new public methods, which you're probably not going to do when creating an MVCI View.  

The other way, when all your doing is "configuration" of an existing `Node` sub-class, is to use a builder.  Happily, JavaFX has an interface called `Builder<T>` with a single method called `build()`.  If you follow this approach, and it's usually the correct one, then you won't have a View class in your framework, but you'll have a ViewBuilder which implements `Builder<Region>`.

# What Goes Where?

One of the nice things about MVCI is that **all** of the decisions about where to put your code are straight-forward and easy to resolve.  Let's look at some of the most important items:

## Business Logic

This is the question to ask first:  "Is this business logic?"  If your application isn't a business application, then substitute some other term like, "game logic".  You get the idea.

If the answer is "YES", then it goes in the Interactor.  No exceptions.

## Presentation

Another question to ask is whether the code deals solely with how the data is presented to the user.  That "solely" is interpreted very narrowly and if the answer is "YES", then it goes in the View.  Is something going to be displayed in a particular colour?  That colour choice is probably solely presentation and belongs in the View.  However, if that colour choice is based on something that smells like business logic, then it's not solely presentation.  You'll need to split out the business logic, put that part in the Interactor and use the results of that to determine the actual colour to be used in the View.

For example, imagine you have a list of grocery items and you want veggies in one colour, and fruit in another, and dairy in another colour.  If your list is just; bananas, carrots, apples, tomatoes and milk, then you'll have a problem determining the colours in the View.  What's a "tomato"?  Is it a fruit, a veggie or dairy?  That's a business logic decision, and cannot be made in the View.  Update your Model such that the list also contains the type, and then populate it accordingly from the Interactor.  Now you'll get; bananas(fruit), carrots(veggie), apples(fruit), tomatoes(technically fruit, but used like a veggie), and milk(dairy).  In your View, you look at the types and determine the colour based on that, because now that is solely a presentation decision.

## Service Logic

It's important to understand that "Business Logic", as it is used above actually means, "business logic specific to this MVCI construct".  This does not include calling external API's or parsing JSON data.  Things like that belong in a layer below your Interactor that holds "Services".

Let's take a look at the fruit and veggie example above.  If your list of items is coming from a database somewhere, you are NOT going to SQL statements or any equivalent in your Interactor.  Create a `GroceryListService` that does the database access, then parses the resultant JSON data and creates a list of `GroceryItem` which probably has at least `name` and `quantity`.  `GroceryItem` is your domain object, and that list is passed back to your Interactor.  

But what about the type?

Let's say that you've found some web service like [Chomp](https://chompthis.com) that can give you information about grocery items, (full disclosure, Chomp doesn't seem to tell you what department stuff is in) and you are going to use it to determine fruit, veggie or dairy.  The code that contains the API calls and the JSON parsing for that function also goes into a service.  However, the code that loops through your list of grocery items, calls the service to find out what type of item it is, and then integrates the two together into the Model **is** business logic for this MVCI construct and goes in the Interactor.

## Presentation Data

The Model contains just "Presentation Data".  In other words, every element is something that is potentially used directly by the View.  
