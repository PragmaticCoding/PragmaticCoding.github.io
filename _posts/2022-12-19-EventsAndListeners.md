---
title:  "EventHandlers, Listeners and Bindings - What to Use Where"
date:   2022-12-19 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/events_and_listeners
Diagram: /assets/posts/MVCI.png
DemoStart: /assets/posts/MvciDemoStart.png
DemoRunning: /assets/posts/MvciDemoRunning.png
DemoDone: /assets/posts/MvciDemoDone.png
excerpt: Taking a look at how EventHandler, Listeners and Bindings are different, and how they should be used in a JavaFX application.
---

# Introduction

One issue that seems to come up quite often is connecting pieces of your application through JavaFX

# State vs Events

Being very, very, very general - and not even directly related to JavaFX - there are two approaches to connecting changes between applications or parts of an application.  One is to track things that happen, Events, and the other is to connect through State.  

Also being very, very general - and still not directly related to JavaFX - it's usually safer to connect via State.

Why?

Because Events are, by their nature, ephemeral.  They happen and they are gone, and if you don't capture them when they happen your just out of luck.  All Events need some sort of communication structure, or bus, to travel on, and every system that needs to track Events needs to be connected to that bus at the time that the Events happen or they won't be aware of the Events.

On the other hand, State can be checked at any time and it will always give the correct answer.  This means that a system (or subsystem) can query another system about State at the time that it needs the data an get the current answer.  This is independent of whether changes to that State had triggered an Event, or whether that Event is still sitting on a bus waiting to be captured, or even if there are many Events sitting on the bus related to this data.  

For JavaFX programmers, the good news is that JavaFX provides an amazing "State Machine" with all the features that you need to connect parts of your application via State.

## Let's Get One "Issue" Out of the Way

Yes, JavaFX is essentially "Event Driven", and "State" is just an illusion.  

Deep, deep "under-the-hood", JavaFX is using Event methodology to provide the actions that enable a State Machine.  Look down far enough and you'll find `InvalidationListeners` running everything State related.

But, it's probably safe to assume that the core JavaFX libraries are robust, well designed and thoroughly tested and can be treated as essentially foolproof.  We can ignore the underlying Event Driven nature of JavaFX and treat its State features as State - and think of them that way.

# "State" in JavaFX = Bindings

## All of Your JavaFX Data Should be Encapsulated in Observables

# "Listeners" Convert State Changes to Actions

# EventHandlers Trigger Actions
