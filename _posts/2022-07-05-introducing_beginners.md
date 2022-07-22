---
title:  "Introducing the Absolute Beginners Guide to JavaFX"
date:   2022-07-05 12:00:00 -0500
categories: javafx
logo: /assets/logos/LittleBrain.jpg
screenshot1: /assets/elements/SceneSwapShot1.png
screenshot2: /assets/elements/SceneSwapShot2.png
screenshot3: /assets/elements/SceneSwapShot3.png
screenshot4: /assets/elements/SceneSwapShot4.png
screenshot5: /assets/elements/SceneSwapShot5.png
excerpt: Introducing a "course" for programmers just starting out with JavaFX.  Everything you need to know to get from absolute zero knowledge to building real applications that do real work.
---

# Why?

Most of the tutorial articles I've written here have been of the "deep dive" nature. I try to go way beyond the information available in the JavaDocs, and to give some insight into how to get the most out of the tools that JavaFX provides. These articles tend to be on the long side, Jekyll tells me that they average about 13 minutes of reading, and there's usually lots of code examples and pontification about clean coding and on and on.

I realize that these are going to be pretty advanced for someone who's questions are about how to get that first screen up and running, or how to swap scenes with FXML.

I've also had the honour to help some students on Reddit with their assignments lately.  While I've tried to be very careful about not doing their homework for them, I've tried to send them in the right direction to build good projects.  I've been surprised at how many times I've been able to say, "Go and read this article that I wrote about this topic, it should tell you what you need to know".  

But, like I said, these tend to be more advanced articles with maybe more information than a student needs to read.  I think it would be nice to have something aimed at people that don't need the "deep dive" articles, just some basic information to get them started.

So I've been working for some time now on a series I'm calling, "The Absolute Beginners Guide to JavaFX". It's all about how to build Reactive applications in JavaFX. Even if you're not a beginner, you may find it of interest.

You can find it here: https://www.pragmaticcoding.ca/beginners/intro

# What's in it?

This is a huge content drop for me, and it's kept me busy for a long time. It's also an ongoing project, so there's more to come. So far, it's divided up into two parts:

-  An introductory application.Pretty much "Hello World" in JavaFX. Then we add some user interaction and some styling. Just enough information to get a beginner started in the right direction.

-  A CRUD application.This is a step-by-step development of a basic Create-Retrieve-Update-Delete application for a customer database. The database is simulated as a simple Java class, and allows the whole application to work just like it would in real life.

At this point, I've completed the development of the CRUD application up to the point where it's a completely functional "Create" application for account number and name. Complete means that it has validation in the GUI, background processing for the save function and handles exceptions from the database. It really is complete, just doesn't have a lot of fields on the screen.

# Programming Approach

First and foremost, this series is about writing *Reactive* JavaFX applications.  So yes, we'll be talking about HBoxes, and TextFields and Labels and all that good stuff, but the focus is on application architecture and Reactive programming techniques.

I should also point out that the content here, while primarily aimed at showing JavaFX concepts, is also aimed at showing good programming techniques, and how to constantly apply established principles to create clean code.

# When will there be more?

I was concerned about finding the right time to publish the parts that have been completed.  A series like this has to have enough content to be of use, but it's pretty big, so you don't want to wait until it's all done before publishing it.  

When you do publish, there's an implied promise to supply more in a reasonable time frame.  Because you don't want to leave people hanging half way through a project.  So I'm going to try to keep putting up more lessons on a regular basis.

As it is, it stops at a logic place, with the completion of the "Create" section, albeit with a limited number of fields and something less than a stellar look and feel.  In the future, I'll publish each part as soon as it is completed which will keep a steady stream of new content, but it might end with some "cliff-hangers" as it goes on.

# What Will Be in the Next Lessons?

The next steps will be to add some more fields, work on some more sophisticated formatting for the GUI and then to add in the "Retrieve" function.

# The Companion Project

There's also a companion project on GitHub with all of the code for every single article. So you don't have to do anything more than download the project to try things out for yourself.

I should also point out that the content here, while primarily aimed at showing JavaFX concepts, is also aimed at showing good programming techniques, and how to constantly apply established principles to create clean code.
