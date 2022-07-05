---
title: Moving On - Building A CRUD Application
short_title: Introduction
permalink: /beginners/crud
structure: /assets/beginners/AppStructure.png
excerpt: Now we are ready to start building an application that actually does something.  We're going to build a CRUD (Create, Retrieve, Update and Delete) application for a customer database.  
---

# Introduction

So far we've built an application that does little, but demonstrates a lot.  Now we're going to write one of the most common types of application out there - CRUD (Create, Retrieve, Update and Delete).

Our example is going to be something super common, some sort of a customer database where we can add new customers, look them up in a number of ways, and then modify or delete them.  

# Software Architecture

Most software applications have a structure that looks something like this:

![App Structure]({{page.structure}})

The top three horizontal boxes (in green hues) represent the GUI elements of the application.  Individual Labels, TextFields, ComboBoxes and such are at the top, then actual layouts that contain them.  The bottom layer of the GUI is the part that contains the mechanics.  These are the things that control actions from the GUI, instantiate the GUI and, of course, the State.

The blue layer is the business logic layer.  This is the part that contains the logic which is specific to whatever function the GUI supports.

Under that, in pink colours are the "external" elements that connect your application to other applications, the persistence layer and calls to external API's.  The "Brokers" layer is responsible for fetching data from these other services and packaging the results into "domain objects", which can then be dealt with by the Business Logic layer.

## Features and Scope

The most important thing, is that when you develop an application you implement one feature at a time, and feature **always** cuts a vertical slice through the application structure - from screen widget right through to external API call.  When you expand the scope of a feature, you are cutting a wider slice through the entire application structure.

What this means is that you can't just add a TextField to a screen and not create the rest of the structure to handle the data in that TextField right down to your database.  On on the other end, if you're also creating the database, you can't add fields to it without creating the Brokers and DAO's and business logic to pass it to the GUI and put it on the screen.

You build your application by starting with a single feature, no matter how small, and implement it from top to bottom in the application structure.  And all the way, you stick to YAGNI (You Ain't Gonna Need It) as your guiding principle.  This means that you ONLY build things directly required to implement the feature that you're implementing.  No peeking ahead.  

As you continue to build your application, you can expand the scope of a feature, or add new features (very often this is just a matter of semantics).  All the while, making sure that you build out each feature from top to bottom of the application structure, and only building the things you need.

This is the process we're going to follow to create our Customer Database application.

# Testing

The unfortunate truth is that GUI's are problematic to test.  There are automated tools, but they are complicated and difficult to set up.  The best thing to do is test by hand.

However, the rest of your system can, and should, be tested, and if your architecture allows for a clean separation between the GUI and everything else (and ours will), you can do unit testing easily.
