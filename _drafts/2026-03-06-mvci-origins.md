---
title:  "On the Origins of MVCI"
date:   2025-10-20 12:00:00 -0500
categories: homelab
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/mvci-origins

JavaDocOL: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/ObservableList.html
JavaDocFXC: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/FXCollections.html

MvcExample1: https://softwarepatternslexicon.com/java/user-interface-design-patterns-in-java/model-view-controller-mvc-pattern/implementing-mvc-with-javafx/
MvcExample2: https://learn-it-university.com/mastering-javafx-implementing-the-model-view-controller-mvc-architecture-in-your-projects/
MvcExample3: https://codezup.com/building-scalable-desktop-applications-with-javafx-guide/
MvcExample4: https://codingtechroom.com/question/understanding-mvc-design-pattern-with-javafx
MvcExample5: https://peerdh.com/blogs/programming-insights/building-a-responsive-user-interface-with-javafx-and-mvc-architecture

excerpt: "A short explanation of how Model-View-Controller-Interactor came to be, and how the components were named."

---

# Introduction

Recently I had a an email conversation with a programmer who had been reading this site and learning about MVCI.  He seemed to have a good handle on how it worked, but was confused about my seemingly random use of the terms "Model" and "Presentation Model" throughtout many articles.  Where these two different things?

I think this is a reasonable question.

One of the few things that I wish I had differently with MVCI was the component names, and getting called on it made me realize that I've never explained how MVCI came about and how the naming was determined.  I never set out to create a framework, I was just trying to get work done.  At some point I realized that I had developed an approach that generic utility, and was something that others might also find useful.

Here's how that happened...

# My Previous Experience

In 2011 I had been working as a professional programmer for about 25 years, and most of that was involved development in variation of what started out as the Pick Operationg System.  Pick was invented in the 1960's as an operationg system, but by the 1980's most of the implementations where integrated database environments running under some other operationg system.

In a nutshell, Pick systems store data in link-hash filing systems implementing key-value pairs.  This works very much the same way that `Maps` in Java work, but for disk storage.  The values are variable-length delimited records that are essentially strings but are divided into fields, which can also be divided into values, which can, in turn, be dividide into subvalues.  For this reason, the modern term for these kinds of databases is "Multivalue".

Multivalue database also include a TCL facility, a query language, a batch processing language and a programming language.  The programming language has traditionally been some version of "Pick BASIC", which looks somewhat like MS BASIC but has a lot of strange contstructs that integrate it with the mulitvalued database.  

In a way, you can compare being a Multivalue developer in 2011 to being like a COBOL programmer.  The languages are from the same generation and far more programmers are retiring or dying of old age than are entering the field.

## The Environment I Was Working In

The company that I had worked for since 1996 had recently aquired one of these Multivalue systems (which was why I was hired), and I had been working with it ever since.  The original version was "green screen" - meaning it ran on a terminal in text mode - but towards the end of the 1990's the vendor had released a client-server version that the client in Windows.  The client was written in Visual Basic 5.  It took them years to write, and VB6 had already come out before they were done, but they didn't want to go and rewrite it before it was released.

When we switched over to the client-server version, we purchased the rights to the source code and we started to do our own maintenance and customization.  Soon we had almost no interactions with the vendor who was bought-out, merged, sold off, merged, bought-out and eventually disappeared entirely.  

This application was huge, I wouldn't have been surprised if it had 2 million lines of code and the server-side was mature, stable and time-tested.  The earliest code comment I ever saw was from 1981.  There was lots of code that we never ran because this software was written for a variety of clients in a particular industry in both Canada and the USA, and there were features that we just didn't need.  But you still had to deal with the code for it.

We became really good at integrating this system with other systems, including our various web servers and business partners.  In fact, in the early 2000's we were probably running way at the forefront for doing these kinds of things - which would have surprised most people given the technology we were using.

We still had a VB programmer to do customizations to the client side.

But in 2011 we were starting to get more than a little worried that the niche nature of running on a Multivalue platform was becoming a significant business risk.  Remember, this was the core application that ran the entire company.

So we decided to start to "future-proof" our technology.  We had a strong demand for new desktop features, but there was no way that we were going to build those in VB5.  We decided to look at Java.

## Starting With Java

The idea was that we would develop as much new business logic as possible into the Java environment.  Whenever feasible, we would avoid creating new Pick BASIC programs, but would implement the business code in Java, with only utility type programs running inside the database server to support the Java code when necessary for performance.

Desktop GUI's would be written in Java.

Of course, I had a small shop of programmers with tons of domain knowledge and programming experience in Multivalue systems, but no Java experience at all.  So I contacted a training firm, booked a boardroom for a week, and we did a crash course.  I alse hired an experienced Java consultant for a one year contract to act as a mentor for the team.

And off we went.

This was literally like going from "Hello World", to building insanely complicated, enterprise critical, multiuser applications in a couple of weeks.  Our first GUI's were written in Swing.

It was nuts.  One of our first screens was so badly written that it literally over-heated the consultant's MacBook.  

We kept going.  What choice did we have?  The challenge was to get proficient in the Java language, OOP concepts, GUI design and Swing while actually producing useable functionality for the company in a reasonable time-frame.  

In 2014, we switched to JavaFX.  I'll be honest, Swing felt tired and had an out-dated look and feel in 2014.  We switched to JavaFX because we just thought it would give a more modern looking end result.  

# Learning JavaFX

We switched to JavaFX as soon as we thought it was "safe".  This meant waiting for the first major update in Java 7 (I think), so Java 7u110 or something like that.  April in 2014 what I remember.

By this time our consultant "mentor" had completed his contract and moved on, and I had hired a new, experienced Java programmer to be our Java Lead.  He was a really smart guy with a PhD, and he was the right guy to have as we muddled our way through JavaFX.

You have to remember that at this time JavaFX was really, really, really new.  You could Google around all day and you'd be hard pressed to find much in the way of tutorials or technical advice about JavaFX.  Everybody, everywhere, was still learning.  Most of the people who posted tutorials didn't know much more than we did.  We were pretty much on our own.

By fall of 2014 we had made the decision to ditch FXML and code our screens by hand. I haven't used SceneBuilder since then.

This may seem like a strange decision, but we did already have experience coding layouts with Swing, so it wasn't a totally new concept to us.  Our experience with FXML was that we wasted countless hours, days, even weeks fiddling around trying to make a layout work or do the simple things we needed.  It was pure frustration.

Some things I learned about SceneBuilder:

It's not a beginners' tool.

: There's nothing in SceneBuilder that guides you through the process of designing a layout.  You have an endless array of choices with both the `Nodes` themselves and the `Properties` inside them.  SceneBuilder gives you zero guidance as to what they mean or how they should be used.

The "Preview" mode of SceneBuilder was less than useful.

: We found that you could get a preview looking good, but when you actually ran the layout in the application it often differed in critical ways.  Trying to adjust parameters to make it work was a good way to waste even more time.

FXMLLoader = "Black Magic"

: No beginner ever really understood what FXMLLoader did or how to use it properly.  Trying to create a layout that was contained inside another layout involved a level of wizardry that I never achieved.

We found that once we shifted away from SceneBuilder it was far easier to understand how the `Nodes` really worked and how they went together.  Hours spent staring at the JavaDocs for JavaFX actually started to make sense when we were hand coding layouts.

# Incrementally Improving Techniques

As you would expect, the code that we produced at the beginning, trying to do things that were way beyond our grasp at the time, was absolutely horrifically bad.  Just awful.  But it ran, and it mostly worked.

My nature as a programmer has always been to spend time thinking about the process of programming.  Every time that we would start on a new aspect of our work, I would try to understand (as best as I could at the time) what when wrong before, and then try to adjust things going forward.

The result was an ever-evolving "rule base" that we used to guide our development.  A lot of these rules have turned out just to be common wisdom nowadays.  But when we started out, they were ground breaking for us.

I remember having a debate with one of the team about how to create a custom `Node`. It seemed to me that it needed to extend `HBox` because that was what it was based on. But she pointed out that meant that someone else could just reach in and alter it in their layout code, because that what you can do with `HBox`.  I don't think we came to an understanding about an adequate solution.

Now it's obvious.  Create a `builder` for the widget using `HBox`, but then return it as a `Region`.  You're not extending any classes at all, and `Region` protects its contents.  The well known rule today is, "Don't extend when all you are doing is configuring".

## Gather Screen Data Together into an Object

One of the first conventions we adopted was to start putting all of the bits and pieces of data that we needed for a screen into a single object.  Then you can pass around all that data together as a single item.

## How to Link Multiple Nodes to Data

Another rule we came upon early was the case where you have several `Nodes` that depend on the same data element in some way.  

Let's say that you have two `Buttons`, and that only one of them should be enabled at any given time, which is dependent on one particular data element.

Now, you have two choices.  Perhaps the most obvious is to link the `Disable Property` of one of them to the data element, and then link the `Disable Property` of the second to the `Disable Property` of the first via `not()`.  It makes sense.  The description was that "only one of them should be enabled at any given time", so linking them to each other seems the clearest way to do that.

The second method is better.  Link the `Disable Property` of both to the same data element, but use `not()` for one of them.

Why is this better?

There's a couple of reasons.  The first is that the description, "only one of them should be enabled at any given time" isn't technically correct.  It's what it looks like on the screen, but it's not *really* the criteria, is it?  No.  The criteria is that `Button A` should be enabled under some condition, and `Button B` should be enabled under some other condition. The fact that those two conditions are mutually exclusive is, at best, coincidental.  **Program it the way you mean it.**

The second reason is that the two `Nodes` have no real reason to know about each other.  Why would you link them together?

It turns out that this second reason is absolutely critical to writing good layouts.  Coupling is the single factor that kills the maintainability of systems, and coupling inside the layout is as lethal as any other coupling.

## Avoid Directly Linking Nodes

An outgrowth of this last rule was the next rule.  What if you *do* have two `Nodes` where the requirement actually is that only one of them is to be enabled at any given time?  

The answer is to create a field in your layout code to hold `BooleanProperty`.  Bind the `Disable Property` of both of your `Nodes` to this field. Now, the two `Nodes` are dependent on this field, and not each other.

The key point here is that you've linked them without creating dependencies on the implementation of either `Node`.  You can change the nature of each of these `Nodes` and you'll never have to worry about the other one.


## Stop Loading and Screen Scraping

The obvious approach to building a screen is to have some kind of method called something like `loadData()` that accepts your object holding the screen data and visits each screen `Node` to update it with the approapriate data.  Conversely, you'll have a routine, probably named `saveData()` that visits each screen `Node` and extracts the data from it, loading it back into your data object.

That's what we did for the longest time.  I think we reached the height of absurdity after one of the programmers learned that "screen data is not domain data" and started writing routines to convert DAO POJO's into JavaFX `Property` POJO's.  We'd have a DAO "Customer" object that had `String` fields like `lName` and `fName`, and the routines would dutifully copy them into screen data with `StringProperty` fields, `lName` and `fName`.  Then we'd use `fName.get()` to load the data into some `Node` in the layout.

What was the point?

We really scratched our heads over this one.  There was no argument, domain objects don't get near the screen.  But how does this help?

The answer turned out to be easy.  Stop using `Node.setValue()`, or `Text.setText()`.  Use `Node.valueProperty.bind()`, and `Text.textProperty().bind()` instead.  Bind them bidirectionally for editable `Nodes`.

When you do this, the screen `Nodes` are now kept synchronized with the values in the data object, and visa versa.  No more screen scraping because the there is only one copy of the data, and it's shared between all the locations that use it.

# Reactive Programming

This last rule was a turning point because with it, we began to do Reactive programming. And we didn't even know it.

Seriously, I'd never even heard of Reactive programming until after I was retired.  That mentor consultant I mentioned earlier had turned me on to the possibility of doing some consulting in React, which I'd never used before.  So I went through an online crash course on React.  

The course started off with the usual stuff, how to build the layouts and syntax and routines you need to call.  That kind of stuff.  Then it got to the module with THE BIG IDEA behind React.  The idea that you have a data representation of the "state" of your GUI, and when you change values in it using a specific process, the GUI will be rebuilt to reflect those changes.  This course really hammered away at this concept like it was going to be extremely hard for a typical programmer to wrap their head around.

And I'm thinking, "This is pretty much the way that I've been using JavaFX for years".

I did a bit more research and it turns out this is why React is called "React".  Because it's a Reactive GUI environment.  

What makes React different from JavaFX (apart from the fact that it's a web technology), is that with React it's the layout code itself that is reactive.  In other words, changes to the state object trigger parts of the layout code to be re-run, but with the new values from the state object.

In JavaFX, the layout code is generally only run once.  The layout itself behaves dynamically in response to changes in the state object through `Bindings`.  

In React, if you want a screen element to be hidden, you'll use an `if` statement in the layout code to look at some aspect of the state object and skip adding it to the layout.  In JavaFX, you'd generally bind `Node.visibleProperty()` and `Node.managedProperty()` to that same aspect of the state object.  

For what it's worth, most Reactive systems use the same approach as React.  Kotlin's Compose for Android apps does the same thing.

Personally I prefer the JavaFX approach.

## The Benefit of Reactive Programming

I have found that Reactive programming simplifies every aspect of GUI design except one situation which it makes slightly more complicated.  Let's look at the simplifications first:

"Set It and Forget It"

: This is amazingly important.  With Reactive design you instantiate a `Node`, you configure it, and you bind some of its `Properties` to `Properties` in your state object, and you place it inside a screen container.  Then you're done with it.  Like, really done with it.  You'll never, ever, never need to reference it again.  You don't need to assign it to a variable, you don't need to give it a name, you don't ever put it in a field.

: And think about it.  If you don't have a variable reference to it, how can you couple to it??? If you want the data inside it, just look at the state object field that it's bound to.  If you want to change it, change a state object field that's bound to one of its `Properties`.

No More Loading and Scraping

: I think this has been covered.  But just to hammer it home, you don't have field references to the `Nodes`, so you aren't even tempted to do this.

No More Passing Screen Data Around

: This takes a little bit more thought to understand how important it is.

: Most of the time, when you do screen scraping to implement that "Save" `Button`, or to perform an action when the value in a `CheckBox` changes, or whatever, you don't want to go through the effort of pulling all the data out of all the `Nodes` to update the state object to send it back to your back-end logic.  So you'll pull a few relevant pieces out of the screen `Nodes` and pass those to the back-end.  And you might have quite few of these actions you need to implement.

: But Reactive programming introduces the idea of a *shared* state object to which both the back-end logic and the GUI hold a reference. Changes to the GUI are instantly reflected in the state object and are instantly available to the back-end logic.  In the same manner, changes made to the state object by the back-end logic are instantly reflected in the GUI without explicitly passing them around.

: So now, when the "Save" `Button` is clicked, the only thing that the GUI needs to do is to notify the back-end logic that it needs to perform "Save" - whatever that is.  The GUI doesn't need to be passed the new data, because it's already in the state object and it can see it.

The Back-End Can Monitor GUI State

: If the back-end logic needs to respond in real time to data changes in the GUI it can do it by monitoring the state object and acting when required. Let's say that you have some complicated filter function that updates the screen as the user types in a `TextField`. No problem, the back-end logic can implement a `ChangeListener` on whatever state object `Property` is bound to the `TextField.textProperty()` and update the filter without any direction from the GUI itself.

### The one thing that is more difficult?

Detecting changes.  This one had me stumped for a while.

If you only have one copy of the data, and you are not using the screen `Nodes` as storage for the "working" values of the data, how do you detect if it has changed?

This can be important because you often don't want that "Save" `Button` to be enabled unless something has changed.  Sometimes you don't want to perform some expensive operation unless the controlling data has changed.

One of my newer programmers ran up against it and did some web searching.  He found a project called 'DirtyFX' by Thomas Nield. Simply put, it's a set of `Properties` that implement the idea of a "baseline" value and present an embedded "dirty" `BooleanProperty` that is true whenever the current value of the `Property` differs from the baseline value.

It's a really clean idea and, as far as I can tell, has minimum overhead compared to using `SimpleProperty`.  As a matter of fact, you could happily just use the `DirtyFX` version instead of the `Simple` version for every `Property` in you application and it wouldn't hurt anything.  There's even a "composite" dirty `Property` that reflects whether a collection of `DirtyFX Properties` contains any changed values.

When we found this library, it was the concrete proof that we were on the right track.  Especially since it was written by somebody like Thomas Nield.  You don't need this unless you only have one copy of the data - the one in the state object.  And you only have that situation if your doing Reactive programming.

# MVC

Right at the beginning our mentor consultant introduced us to the idea of frameworks like MVC and MVP.  Looking back, I can say that we didn't really get it.  We tried to follow the forms, and our applications did have all the parts but it just wasn't quite working.

One of the issues that we had was that all of the on-line tutorials about MVC back around 2017 and for years after that were really horrible.  They talked about the components and then the examples just didn't follow the rules.  It's even harder if you're not using FXML, because all of the examples do, and the hopelessly muddle up the MVC Controller with the FXML Controller, and then they just put everything that's not state or layout into the FXML Controller.  This includes application logic and back-end file or database access.

I've just done a search, and the results look a bit better today.  

First of all, on DuckDuckGo a lot of PragmaticCoding pages come up - but this may just be some kind of search personalization even though DuckDuckGo isn't supposed to do as much of that.  But take it as a win, anyways.

Secondly, a lot of the examples pay lip service to the idea of having the Model do the file access.  Yeah! That's a big improvement.  Take a look at [this tutorial]({{page.MvcExample2}}). It doesn't have any application logic other than `loadData()` and `saveData()` skeletons, but it's more than these examples used to.  It's also pretty typical of what you'll see in a modern tutorial.

I do see a depressing number of web pages that are so simple they are almost "content free". Take [this page]({{page.MvcExample4}}) which has a 20 line example that, while it doesn't do things the way I would do them, has all the parts of MVC and has them used properly.  However, it doesn't actually *do* anything, and there's application logic in the Model because there's no application logic in the application.  An example without any application logic in the Model is virtually useless, because it leaves out a vital aspect of the MVC architecture.

Here's [another tutorial]({{page.MvcExample5}}) that is way more complex, yet still avoids any application logic.  On top of that, it attempts to implement the FXML Controller as the MVC Controller.  The only code that comes remotely close to application logic is its `handleSubmit()` method in the FXML Controller.  This method should just call a method in the Model, but instead manipulates Model fields itself.

Finally, there's [this guide]({{page.MvcExample3}}) that has a lot more meat on it.  Once again, there no actual application logic but there is an example of asychronous processing that just does a `sleep` in a `Task`.  This is inside a class called `LongRunningTask`. It's called from the FXML Controller, which, once again, is acting as the MVC Controller.  Now, there's nothing wrong with calling `LongRunningTask` from the MVC Controller **if** all it does is call a method in the Model, but since all we have is a `sleep`, we are missing essential guidance here.

I was unable to find any example that included actual application logic contained within the Model beyond simple save/load skeleton methods.  

And back in 2014 the results were even worse.

## Misunderstanding MVC

I think it would have been a miracle if I had actually figured out the correct way to implement MVC with JavaFX back then.

And that miracle just didn't happen.

I thought that the "Model" in MVC was simply the screen data.  And I thought that "Domain" simply meant the domain of the whatever the View was displaying.  

But I still had the same question... Where would the business logic go?  

I did not like the idea of stuffing it all in the Controller, so I wanted to split it out.  I also had this idea that a class who's methods were only ever called from one other class wasn't really actually a separate class - an idea that I stronly disagree with today.

# Enter the Interactor

You have to remember that right from the start we were building applications designed to work with a complex mission critical multi-user system. A system that processed thousands of transactions moving millions of dollars every month.  We had to do it right or it would be really, really bad.

We established one unbreakable rule when we first started righting Java applications, "Read anything you want directly from the database, but if you are updating legacy data you need to do it through the legacy system".  That meant calling the server-side routines written in Pick BASIC.  

Some of the subsystems maintained audit logs and detailed histories of data changes.  Updating them meant using the that subsystem's transaction processor.  This meant that you had to initiate a persistent connection to the database, run some rountines to initialize a transaction, run some more to load data into the transaction, some more to update the data in the transaction from your application, run some more to post the transaction and verify the results, and then finally some more to close the transaction and terminate the database session.

In other words, you didn't just update the legacy system, you truly had to **interact** with it.

I'm not sure we put all that much thought into it, but a huge part of "application logic" for us was interacting with the legacy system. So we called this the "Interactor".

I still like this name.  Even if it's not interacting with other parts of the system, it's the place where the domain data and the view data interact.  So it works.

# Presentation Model

So far I've avoided using the term "Presentation Model" outside of the introduction.  I've stuck to variants on "state data object" instead.  

You can see that by misunderstanding the structure of MVC, I had shifted "Model" into an entirely d
