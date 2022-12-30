---
title:  "Another Reason to Get Off the FXAT"
date:   2022-11-15 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
ScreenSnap: /assets/posts/StarterFX.png
GitHubDialog: /assets/posts/StarterFX-GitHub.png
Diagram: /assets/posts/MVCI.png
excerpt: Perhaps the most important reason to learn how to deal with the FXAT is build a different way of thinking about your GUI code.
---

# Introduction

# A Story

When I started learning Java we were building GUI applications to extend our legacy system.  At the time, we were using Java Swing, since JavaFX hadn't yet been released (in its current form).  One of the first things that we built was some utility to do some complicated change to a customer account.  Done manually, it took the Customer Service department a long time to handle these requests, so it seemed a good place for us to start.

We had one of our most experienced Java programmers working on the front end for this.  Essentially, it was a screen with a few options, mostly using ComboBoxes (whatever they are called in Swing), a place to pick the customer account and then some kind of an area to show the progress and the results of the processing.  And a button that said, "Go", or something like that.

The processing took a fairly long time to complete.  It was 12 years ago, so I can't remember exactly, but probably in the region of 30-60 seconds.  It was an involved process with multiple steps and required lots and lots of requests back to the database, and the legacy application API.  The legacy database schema was ancient, and certainly not designed for the kind of client-based computing that we were doing.  So we needed to grab a record, do some stuff with it and then figure out what the next record would be, and then grab that and so on.  

Over the course of years, we'd figure out better ways to work around these issues so that we didn't have to asynchronously process records one at a time.  But back in the early days, this was what we had.  It took about 80-100ms to process a request through the database, so if you had to make a 100 calls that would be 10 seconds right there.  So process that took 30-60 seconds wasn't great, but it wasn't totally unreasonable at the time.

This programmer had written the code that was invoked from the GUI and he'd put lots of places in it where he'd update the GUI with the progress of what was happening.  Because there's nothing worse than having a GUI that sits there with a blinking cursor while it works, looking like it's hung.  But that's exactly what happened.

I remember vividly what he said in a meeting one day, frustrated with how it was misbehaving:

> "I don't understand, I'm posting the updates to the screen but it's not changing.  Then when the process finishes, all the updates to the screen happen all at once."

Even back then, I knew what the problem was right away.  He was doing the processing on the EDT (Swing uses something called the "Event Dispatch Thread" instead of the FXAT).

But that's not really enough to understand what was happening.  He was, after all, updating the GUI from the EDT - so it should have happened right away, no?

It was a long time ago, and I don't have access to the code, but I have a couple of ideas:

1. The first possibility is that he was updating the data associated with the screen elements and triggering the screen to updates.  But each update is triggered like an Event, and the action to update the screen ends up on the Event Dispatch Queue.  Since his job is currently running on the EDT, and will continue to do so for some time, no new jobs are getting pulled off the Queue and run.  Then, when his job finally completes, all the updates are pulled off the Queue and run.

1. The second possibility is that he was using `SwingUtilities.invokeLater()` (the same as JavaFX's `Platform.runLater()`) to do the updates.  After all, this is what the tutorials tell you to do.  Just that he missed the part about running his big job on a background thread. `SwingUtilities.invokeLater()` puts your job on the Event Dispatch Queue, and the result is the same as the first scenario above.

Over the course of years, we switched over to JavaFX and we had a fair number of new developers pass through the team.  Almost without exception, I'd catch the newer programmers doing stuff on the FXAT that should have been on a background thread, even though we went over it time and time again.

Some of the programmers seemed absolutely determined to find a way to somehow program a long/blocking process like it was a `Function`.  That is to say, input a value and get a value back.  

The problem, though, is what happens in the middle.  It always seems to involve the same concept - "wait".

# Event-Driven Systems and Linear Thinking

Programmers are really good at what I call, "Linear Thinking".  That's the idea that you do one thing, get the results and then do the next.  Rinse and repeat.  Sometimes you do "this" instead of "that", sometimes you loop, sometimes you quit early.  But you do one thing after the other and get to the answer at the end.

Graphically, it looks like this:
