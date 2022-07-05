---
title: Part 11 - A "Working" Application
short_title: Part 11
permalink: /beginners/part11
screenshot_1: /assets/beginners/part10_1.png
excerpt: At this point we have a pretty clean "Create" application.  It's error resistant, doesn't corrupt our database and ready to add some more functionality.  Let's look at what we've done so far.
---

# Reviewing What We've Done So Far

At this point we've actually got a simple, but fully functional "Create" application.  It doesn't have anything more than an account number and a name, but to get this far we had to put together a lot of moving parts.

This is probably a good time to pause, and look at what we've done.

# We've Built on a Framework

At this point the value of building on a framework should be readily apparent.  

Here are some of the things it does for us:

## We Can Use Unit Tests

Because we now have a pretty clear dividing line between the Reactive JavaFX code and the rest of our application, it's now clear that some of our code is just plain old Java code and there's nothing special about it.  That means that we can test it, and that means that we should test it.  

## Our GUI is Just the GUI


## Coupling is Minimized

Within the framework, there's no dependency between the View and the Interactor.  They both deal with the Model, but don't even know of the existence of each other.  

## Everything is Easy to Find

There's no need to go looking for code.  You won't find business rules in the View or the Model, they're all in the Interactor.

## Everything is Small

No 1000 line classes here.  Keeping everything broken up into discrete parts means that all of the parts stay small.

# Simulated Database and Access

# Moving On From Here

Our next steps are going to build on what we've done so far:

Add Another Field
: We'll add a "Phone Number" field to the screen so that we can see how the feature build cuts all the way through the application from GUI to the database.

Clean Up the GUI
: It's going to become clear that our GUI is a bit ugly when we add another field.  In the process of cleaning up the GUI, we'll see how our layout has become cluttered with stuff that isn't strictly...layout.  We'll clean that up and see how an even nicer looking GUI can actually take a much less code in our ViewBuilder.

Split the Name
: We'll divide the "name" field up into three parts including a "salutation" field that will let us use a ComboBox.

Add the Rest of the Fields
: We'll add a bunch of address fields to our screen, and then down through to our database.

Add a "Retrieve" Function
: Finally, we'll be ready to add the next CRUD function, "Retrieve".  Without "Retrieve" we can't do "Update" or "Delete"
