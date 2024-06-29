---
title:  "Tracking Task Progress"
date:   2024-06-20 00:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/task-progress
Diagram: /assets/posts/MVCI.png
Modena: /assets/elements/modena.css
excerpt: What if you need to control the styling of a Node over a range of values?  How can you do that, and is there any way create a framework that you can use over and over again?
---

# Introduction

# Under the Hood With Task Updates

>protected void updateMessage(String message)
>
>Updates the message property. Calls to updateMessage are coalesced and run later on the FX application thread, so calls to updateMessage, even from the FX Application thread, may not necessarily result in immediate updates to this property, and intermediate message values may be coalesced to save on event notifications.
>
>This method is safe to be called from any thread.

What we want to understand here is this part:

>...intermediate message values may be coalesced to save on event notifications.

What does this mean?

Let's say that you have a background thread that's generationg gazillions of calls to `Task.updateMessage()` from a background thread - way faster than the FXAT can process them given all the other stuff it's doing.  Well, in that case, it seems that this means that not all of our calls to `Task.updateMessage()` are going to result in screen updates.  

Let's take a look at the source code for `Task.updateMessage()` to see how it does this, and what it really means:

``` java
private AtomicReference<String> messageUpdate;
   .
   .
   .
protected void updateMessage(String var1) {
    if (this.isFxApplicationThread()) {
        this.message.set(var1);
    } else if (this.messageUpdate.getAndSet(var1) == null) {
        this.runLater(new Runnable() {
            public void run() {
                String var1 = (String)Task.this.messageUpdate.getAndSet((Object)null);
                Task.this.message.set(var1);
            }
        });
    }
}
```
First, we have a variable `messageUpdate`, which is an `AtomicReference<String>`.  `Atomic` variables are designed to be used by multiple threads, which primarily means that get and set functions can be combined into a single, indivisible, atomic, operation.  In this way, it's guaranteed that some other thread won't get in and change the value in between your thread checking the value and then changing it.  

This concept is key to how this works.  First, let's look at the flow when the call to `Task.updateMessage()` comes from a background thread...

To understand this, you need to keep track of what code is running on what thread.  The function is entirely composed of a single `if` statement.  So if we come in on a background thread, then the `if` and the `else if` are going to run on that background thread.  Most importantly, it means that:

``` java
if (this.messageUpdate.getAndSet(var1) == null)
```
including the call to `messageUpdate.getAndSet(var1)` will run on the background thread.  `AtomiceReference.getAndSet()` will instantly update it's stored value with `var1`, and then it returns whatever was previously stored in it.  

Only in the case where the previous value in `messageUpdate` was `null` will it invoke `Platform.runlater()` to update the `message Property`.

Why?  To understand that we need to look at what happens inside `Platform.runLater()`.  There are two lines of code:

``` java
String var1 = (String)Task.this.messageUpdate.getAndSet((Object)null);
Task.this.message.set(var1);
```
The first line gets the value from `messageUpdate` and instanly sets it value to `null`.  The second line will update the `Property` with that value.

From this, you can see that the FXAT is the only thread that puts `null` into `messageUpdate` (well, `var1` could be `null`, but let's not worry about that).  So if our background thread checks `messageUpdate` and finds that it's not `null`, then that must mean that there is a pending operation in the FXAT's queue to update the `message Property` from `messageUpdate`.  In that case, it's not necessary to invoke a new FXAT job, but we can update the value in `messageUpdate` (actually, we already did that) and this new value is the one that the FXAT will pick up.

Let's imagine that we have a `message` that we want to walk through the values, "Starting", "First", "Second", "Third", and "Fourth".  These values to be supplied from our background thread.  We'll assume that `message` has just been updated to "Starting" and it's put `messageUpdate` to `null`.  The background thread is running really fast, and it will call `Task.updateMessage()` a few times before the FXAT can get around to copying the value into `message`.  

Here's what our values look like:

| Thread    | New Value |Old Value | message | Queue FXAT Job? |
|-----------|-----------|----------|---------|-----------------|
|Background |First      |Null      |Starting | Yes             |
|Background |Second     |First     |Starting | No              |
|Background |Third      |Second    |Starting | No              |
|FXAT       |Null       |Third     |Third    | Is the FXAT Job |

From this, you can see that only one job is launched for the FXAT, and at that time, the value of `messageUpdate` was "First".  However, by the time that that initial job on the FXAT was executed, the background thread had called `Task.updateMessage()` two more times, and the value in `messageUpdate` was now "Third".  The values "First" and "Second" never made it to `message`.  Once again, `messageUpdate` has the value `null`, and any call to `Task.updateMessage()` from the background thread will queue up a new FXAT job.

But what if a call to `Task.updateMessage()` comes from the FXAT?  Then it just updates `message` and doesn't touch `messageUpdate`.  But what if `messageUpdate` isn't `null` when this happens?

If `messageUpdate` has a `non-null` value in it, then there is a job on the `FXAT` to copy the current value of `messageUpdate` into `message`.  The FXAT always processes jobs in sequential order.  This means that if this code is running from the FXAT, then it was queued before the call from the background thread called `Platform.runLater()`.  It may be running *after* that background code ran, but the job on the FXAT generated by that background code is still pending processing.  

All of this means that it's pretty easy to implement our own status updates that work the same way..

## Creating Custom Data Updates
