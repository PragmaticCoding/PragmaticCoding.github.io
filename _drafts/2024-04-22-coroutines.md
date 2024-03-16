---
title:  "JavaFX: Kotlin Coroutines"
date:   2024-03-09 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/coroutines
ScreenSnap1: /assets/posts/StarterFX.png
ScreenSnap2: /assets/posts/StarterFX.png
ScreenSnap3: /assets/posts/StarterFX.png

excerpt: Kotlin's coroutines are a useful alternative to Task for blocking operations.
---

# Introduction - Dealing With the FXAT

The one unbreakable rule in programming in either Swing or JavaFX is "Don't perform long operations on the GUI thread".  

But why?

I'm going to talk about JavaFX, but everything here, for the most part, applies to Swing as well.  

JavaFX `Nodes` and `Properties` are not "thread-safe".  This means that they are not designed to handle situations where programs running on two or more threads are simultaneously, or nearly simultaneously, accessing their data - otherwise known as "concurrency issues".  

Imagine that you have an object with a field that is accessed by one or more methods somewhere, for now let's just think about a single method inside the object.  Let's imagine that it's structured something like this:

``` kotlin
public fun doSomething(startingValue: Int) {
   someField = startingValue
   for (x in someReallyBigList) {
      some code that takes a while to run and uses "x" and "someField"
      someField += 1
    }
}
```
It's possible that if a process on one thread started running this code, then another thread could start running the same code a few milliseconds later.  When the second thread started, `someField` would be reset back to `startingValue` and then both threads would be simultaneously incrementing `someField` while in the loop.  

It is possible to use `synchronized` in both Java and Kotlin to lock out the second process until the first one has completed, but even this approach has issues and ends up with much more complicated code.

To avoid all of this, JavaFX restricts access to `Nodes` and `Properties` that are connected to the SceneGraph to a single thread, the "FXAT" (in Swing it's called the "EDT").  With only one thread accessing the objects, there are no concurrency issues.

How does code get run on the FXAT?

Code can be run on the FXAT by executing it through an `EventHandler` or by putting it in a `Runnable` executed through `Platform.runLater()`.  Code executed this way is added to a FIFO queue of "jobs" which are picked up be some master loop in `Platform`.  If there are no jobs, presumably it performs some kind of `sleep()` and loops again.  

What's important here is that even "under the hood" operations in JavaFX, like repainting the screen and recaluclating layouts are all done via the same mechanism - they need to wait to get their turn to run on the FXAT.  The impact of this is that the code that you write to run on the FXAT has finish up in just a few milliseconds.  If it often takes much longer, then you'll back up that queue on the FXAT and the response will get sluggish.

If you write code that takes a long, long time to run on the FXAT then you'll virtually hang your GUI.  


# A Quick Look at Task

# What is a Coroutine?

OK, while it is possible to have an application that does need to do computation that require a long length of time to run, it's far more likely that your "long" jobs are spending most of their time waiting, not working.

Whenever you need to access a file, or connect to a database or call a remote API on the network you are going to encounter latency of some sort.  And that latency is going to be huge compared to the amount of time your code is actually running to do something.  I did some testing of a system I was working on, and I found that once I had a response from the database, all of the Java code that turned the raw data into domain objects completed in 4 or 5 milliseconds.  But I also found that it took about 30-50 ms for a typical database request to complete.  Some domain objects required several, or even many cascading database calls to build, and if I was preparing data to fill a `TableView` I might need to instantiate hundreds of domain objects.  That could easily add up to thousands of milliseconds of waiting.

Typically, whatever library that you're using to get your data, whether from a file system or a remote API, is going to eventually hand off control to the operating system which is going to do something involving disks or networking.  And when that control is passed on, your client code executes a `wait()` call.  When the O/S work is completed, it sends a signal back to the thread that your wait is running on, and it wakes up and carries on working.

That process of `wait()` followed by a signal is called "blocking".  

But Kotlin has a facility called "coroutines", which allows multiple blocking operations to be run on the same thread.  When a coroutine becomes blocked, it is "parked" in a holding area, which frees the thread up to run another coroutine.  When a parked operation wakes up, it waits until the currently active coroutine on the thread ends or is blocked (and gets parked) and then it can run.  Since each of the coroutines probably spends a very small amount of time doing work and much more time blocked, this means that many coroutines can share the same thread with no, or minimal, performance impact.

# When You Can Use Coroutines on the FXAT

Blocking on the FXAT is usually very bad - don't do it.

However, it also turns out that the FXAT works really well with coroutines.  Why?

Remember that description of `Platform` as a loop that runs on the FXAT, using `sleep()` when the queue is empty?  Well, `sleep()` is a blocking operation too.  We have a situation where the main FXAT loop is constantly blocking internally which means that the FXAT can be shared with other blocking code if we use coroutines.

There are some caveats about this.  The nature of your GUI has allow for lots of interal blocking on the FXAT.  If your GUI is super active and spends way more of its time doing stuff and has very little blocking, then it's not going to work very well with coroutines.  Your long tasks need to be long tasks because they are blocking, not because they spend hundreds of milliseconds doing complex calculations.  

# How to Use Coroutines

## Coroutines on Any Old Thread


## Coroutines on the FXAT
