---
excerpt: An Introduction
hexmap: /assets/cybertanks/hexmap.png
permalink: /javafx/cybertanks/intro
---

![CyberTank]({{page.banner}})

# A Full-Blown Game in JavaFX

Welcome to the CyberTanks! series.  This is all about creating a large-scale game based in JavaFX.  It's an adaptation of a game developed in the 1970s-1980s called "Ogre", from Steve Jackson Games.

Like a lot of other war games, it's played on a hex map with counters to represent the various units and their capabilities.

What makes Ogre different, is that one side (or sometimes both sides) has only one unit - a giant cybernetic tank called an "Ogre".  Ogres are so big and powerful that they have an information card that shows all of their treads, missile launchers and guns and you track how they take damage on those subsystems individually.  Generally, the other side has more traditional military units, like infantry, tanks and howitzers which are represented by a single counter each.

# Hex Map

This is a hex map based game.  This means that the playing board is composed of hexagonally shaped areas arranged in rows and columns.  It looks like this:

![Hex Map]({{page.hexmap}})

Hex maps are nice because each location has 6 neighbours, instead of just 4, which makes a difference in war games likes this.

It is, however, a bit more challenging to code up because, while the columns line up, the rows zigzag and, of course, hexagonal shapes are a little harder to draw than squares.

# Technical Stuff

Some notes on the technical aspects of this project:

## Kotlin

I'm going to write this in Kotlin, because...Kotlin!!!!  Honestly, Kotlin is so much more enjoyable to write in than Java that it's worth learning, and once you've learned it, you'll want to use it all the time.  Kotlin is even better with JavaFX because it makes a lot of the boilerplate coding of JavaFX much, much easier to deal with.

Kotlin is close enough to Java that you shouldn't have much difficulty figuring out what's going on in the code samples included with the articles.

## MongoDB

I'm using MongoDB to store the data for the game setup and maps.  Too me, MongoDB (and probably any document database) is like a modern version of the `Pick` database concepts but better suited to integration with Java based languages.  That being said, I'm going to architect the application such that the database implementation is separated from the rest of the logic so that you could swap in another database if you chose.

Additionally, MongoDB essentially stores the data as JSON documents, so the Kotlin serialization to JSON libraries make it super easy to implement.

One last thing about this.  This project is meant to be a demonstration and learning tool and is NOT about creating a finished product that plays Steve Jackson's Ogre game.  So yes, using a database is a bit of a heavy-weight approach to storing game data, but it takes it one very crucial step away from being shippable as a Jar file that someone can just install and run.  And that's very important.

# Approach

This project is going to be built as a Reactive JavaFX application using MVCI (Model-View-Controller-Interactor) as the framework for the components.   

I'm going to try to write this series as the game is being developed.  It's not going to be possible to keep making new versions of the project for each successive article, as I did with the "Beginners Guide to JavaFX", because it's just going to be too big and complicated for that.  So it's likely that code described in an earlier article is going to be refactored and changed later on.  

I'll try to put code samples in the articles so that there will always be a reference version to go along with the text. But be aware that if you compare that to the project from GitHub, there may be differences in the code.  

# Project Management

Even as a solo project, you still need to adopt some kind of project management methodology to keep yourself on track.  I'm going to use what I call the, "Agile in My Head", approach.  Basically, I'm going to keep a Product Backlog (PB) in my head and work down it feature by feature.  I'm going to trust that the things that have come up as issues as I've been working on new features are going to have me grooming and prioritizing my PB as I go along, so it should be clear what needs to come next - at least at first, and probably all the way through.

I'm also going to trust that I have the personal discipline to follow the "Just Good Enough" principle and stop working on a feature when it is just that - just good enough to do what it needs to do.  Embellishments and enhancements can be dropped into my mental PB to be addressed later on, whenever it's appropriate.

I've been practising Agile project manangement for decades.  So most of this kind of thinking is second nature to me and it shouldn't cause any issues.  If you don't have this kind of experience, you should write your PB down somewhere and add enough details to each feature when you get close to building it that you know what the scope should be.  Hold yourself to that scope when you program and add any enhancements that come to you as you program to your PB.  
