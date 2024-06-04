---
title: Part 8 - Dealing With Latency and Blocking
short_title: Part 8
permalink: /beginners/part8
screenshot_1: /assets/beginners/part4_1.png
screenshot_2: /assets/beginners/part8_1.png
excerpt: Our simulated database doesn't really do a good job standing in for a real database because it works too fast.  We'll slow it down and see what horrible things this does to our application.  Then we'll see how to deal with this.
---

# What You'll Learn

1. The importance of using background threads.
1. How to use Task to process on a background thread.
1. Considerations for the GUI while a background task is running.
1. Avoiding concurrency issues when multi-threading
1. Avoiding Button double-clicks.


# The Problem With Our Application

Virtually any database or external API you call from your application has at least the potential to take a measurable amount of time to respond, or to possibly time out and not return an answer at all.  

What happens is that your Java code eventually needs to hand off to some other process to do the work.  Maybe you've executed an HTTP "Get" command, or you've asked the kernel to read a file, or you've invoked an API handler for an application running on your computer.  In all of these cases, your Java code is going to invoke an external process and then execute a `wait()` statement, or at least its equivalent.  When the external process completes, it will send a signal to your Java process to wake up and continue on.

The problem that this presents for JavaFX is that it's always busy.  It's always checking for mouse movements, events that are triggered, button clicks, data changes that impact the GUI.  It's always busy and it all happens on a single thread - the "FX Application Thread" or "FXAT".  This includes the code that you write that works as an EventHandler - it all runs on the FXAT by default (as it should).

The one thing you should **NEVER** do is execute a `wait()` (or its equivalent) on the FXAT.  When you do this, everything in your GUI - **EVERYTHING** - stops dead.  As a matter of fact, you should never execute anything on the FXAT that takes more than a few milliseconds to run.  

Right now, our database is an inaccurate simulation because it has no latency.  Let's fix that...

# Adding Latency to Our Database

The easiest way to add latency is to simple put a `Thread.sleep()` command in our database method. Like this:

``` java
public class CustomerDatabase {

    private Map<Integer, Map<String, String>> data = new HashMap<>();
    private Integer nextKey = 0;

    public int saveCustomer(Map<String, String> customerRecord) {
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        customerRecord.put("_id", (++nextKey).toString());
        data.put(nextKey, customerRecord);
        return nextKey;
    }

    Map<Integer, Map<String,String>> getData() {
        return data;
    }
}
```

If you run this now, you'll see that the GUI goes entirely inactive for 10 seconds.  Nothing happens when you click on a TextField, click the button or try to expand the window.  I'd show you, but there's simply nothing to see.  It's just...dead.

There's a problem with this approach, though.  Our tests now take forever to run.  Let's look at the console output from running the database tests:

``` console
> Configure project :
Project : => 'ca.pragmaticcoding.beginners' Java module
> Task :compileJava UP-TO-DATE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :mergeClasses SKIPPED
> Task :compileTestJava
> Task :processTestResources NO-SOURCE
> Task :testClasses
> Task :test
BUILD SUCCESSFUL in 52s
4 actionable tasks: 2 executed, 2 up-to-date
11:36:56 a.m.: Execution finished ':test --tests "ca.pragmaticcoding.beginners.part8.CustomerDatabaseTest"'.
```

52 seconds!  

This is not acceptable.  Unit tests should never take more than a few milliseconds to run each.  This is one of the reasons that you never access live databases from unit tests.  

We need to find a way to add the latency to the application without slowing down our tests.

One way to do this is to implement the `sleep()` in a separate class extended from our database class.  Then we can continue to test the original database class, while using a "slow" version for our production code:

``` java
public class SlowCustomerDatabase extends CustomerDatabase {

    private final int delay;

    public SlowCustomerDatabase(int delay) {
        this.delay = delay;
    }

    @Override
    public int saveCustomer(Map<String, String> customerRecord) {
        delay();
        return super.saveCustomer(customerRecord);
    }

    private void delay() {
        try {
            Thread.sleep(delay);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

Then we'll modify `CustomerDao` to use our "slow" version for the production code:

``` java
public class CustomerDAO {

    static CustomerDatabase database = new SlowCustomerDatabase(5000);

    public int saveCustomer(Map<String, String> customerRecord) {
        return database.saveCustomer(customerRecord);
    }
}

```

Now our production code runs with a delay, and our test code completes in less than 1 second.  Which means that we've simulated the problem.

# The Solution: A Background Thread

Now, if our database has latency and blocks, meaning that we can't access it on the FXAT, then the correct solution is to do the access on some other thread.  This is usually referred to as a "background thread", because it runs in the background while the FXAT keeps doing its thing.

JavaFX has a tool just for doing this.  It's called `Task`.  So that's what we're going to use.

In MVCI, the Controller is responsible for controlling threads.  So that's where we are going to implement our `Task`:

``` java
public class CustomerController {

    private final Builder<Region> viewBuilder;
    private final CustomerInteractor interactor;

    public CustomerController() {
        CustomerModel model = new CustomerModel();
        interactor = new CustomerInteractor(model);
        viewBuilder = new CustomerViewBuilder(model,this::saveCustomer);
    }

    private void saveCustomer() {
        Task<Void> saveTask = new Task<>() {
            @Override
            protected Void call() {
                interactor.saveCustomer();
                return null;
            }
        };
        Thread saveThread = new Thread(saveTask);
        saveThread.start();
    }

    public Region getView() {
        return viewBuilder.build();
    }
}
```
Amongst other things, `Task` implements `Runnable`, so we can put it into a `Thread` and call `Thread.start()`, which is what we do here.  Really, at this point, all we've done is take the call to `CustomerInteractor.saveCustomer()` and run it in another thread.  We don't need Task to do *that*, do we?

Let's take a look at what happens when this runs.  There's nothing much to see, but the GUI isn't dead any more.  I can interact with it while I'm waiting for the database call to complete, and it works.

But there's a problem.  Here's what I did:

1. Entered account, "123" and name "Fred"
1. Clicked "Save"
1. Clicked "Save" again.
1. Went back to the name TextField and added " Smith"
1. Clicked "Save" again.
1. Clicked "Save" again.

Here's the console output:

``` console
> Task :run
Saving account: 123 Name: Fred S Result: 1
Saving account: 123 Name: Fred Smith Result: 2
Saving account: 123 Name: Fred Smith Result: 3
Saving account: 123 Name: Fred Smith Result: 4
```
Oh, oh!  There's no save for just "Fred"!  The first save is for "Fred S".  At no point did I click when the name TextField said just "Fred S".  What's going on here?

It turns out the reason for this is in the Interactor:

``` java
public void saveCustomer() {
    int result = broker.saveCustomer(createCustomerFromModel());
    System.out.println("Saving account: " + model.getAccountNumber() + " Name: " + model.getCustomerName() + " Result: " + result);
}
```
We call the Broker, which calls the DAO which calls the "slow" database which returns a results after 5 seconds.  Then we output some information to the console which combines the return value from database with data from the Model.  But the Model is now shared between two threads, the FXAT and our background thread.  And when we click over and over, it's shared by even more threads.

This can be a problem.

Since changes to the GUI are "live" changes to State, whenever we need to use State in a background thread we need to do one of two things:

1.  Copy the elements of State to some place private to the background thread.
1.  Lock the GUI so that it cannot update State.

Let's do the first here:

``` java
public void saveCustomer() {
       String customerName = model.getCustomerName();
       String account = model.getAccountNumber();
       int result = broker.saveCustomer(createCustomerFromModel());
       System.out.println("Saving account: " + account + " Name: " + customerName + " Result: " + result);
   }
```

That's an improvement.  Now when I enter "123" and "Fred", click "Save" and then change the name to "Fred Smith", then click save two more times I get:

``` console
> Task :run
Saving account: 123 Name: Fred  Result: 1
Saving account: 123 Name: Fred Smith Result: 2
Saving account: 123 Name: Fred Smith Result: 3
```

That's way better.  It's probably a safe bet that the background thread is going to launch and the broker call is going to start executing before anyone could ever get their hand off the mouse and start typing into any of the `TextFields`.  That being said, this is something that you might want to look out for in a real application.

But it's still disturbing that you could click on that Button three times before it finished processing.  That's something that we should deal with.

## Task Completion

One of the things that makes `Task` better than just a `Runnable` is that it fires an `Event` on completion.  You can set up an `EventHandler` (which will run on the FXAT, as all `EventHandlers` do) to catch that `Event` and do something with your GUI.

Generally, the process is like this:

1. Put the GUI in the state that you want it while the `Task` runs.
1. Execute the `Task`.
1. Put the GUI back into its "normal" state when the `Task` completes.

The first and last steps are performed on the FXAT, the `Task` executes on the background thread.

There's just one problem, though.  The GUI stuff is the property of the View, and the `Task` is part of the Controller.  How do you handle this?

The first thing we're going to do is change our `saveHandler` from a `Runnable` to a `Consumer<Runnable>`.  Then we're going to have our Controller execute that `Runnable` passed through the `Consumer` in the `Task` completion `EventHandler`.  It's more complicated to explain than it is to just see the code:

``` java
private void saveCustomer(Runnable postTaskGuiActions) {
   Task<Void> saveTask = new Task<>() {
      @Override
      protected Void call() {
          interactor.saveCustomer();
          return null;
      }
  };
  saveTask.setOnSucceeded(evt -> postTaskGuiActions.run());
  Thread saveThread = new Thread(saveTask);
  saveThread.start();
}
```

Now `saveCustomer()` is passed a `Runnable`, which it runs when the `Task` completes.  

What happens in the View is a little bit more interesting, first the constructor:

``` java
public CustomerViewBuilder(CustomerModel model, Consumer<Runnable> saveHandler) {
   this.model = model;
   this.saveHandler = saveHandler;
}
```

Next, the code for the Button:

``` java
private Node createButtons() {
    Button saveButton = new Button("Save");
    saveButton.setOnAction(evt -> {
        saveButton.setDisable(true);
        saveHandler.accept(() -> saveButton.setDisable(false));
    });
    HBox results = new HBox(10, saveButton);
    results.setAlignment(Pos.CENTER_RIGHT);
    return results;
}
```

Now, when the `Button` is clicked, the first thing that happens is that it is disabled.  Then the `saveHandler` is invoked, passing it a `Runnable` that re-enables the `Button`.  Like this:

![Disabled Button Screenshot]({{page.screenshot_2}})



In this way, we keep the GUI code in the ViewBuilder, and the control over the threading in the Controller.

# Conclusion

Connecting your application to the real world means that you have to cope with blocking API calls, and that means you have to use background threads to handle these connections.  But multi-threading means concurrency and concurrency brings its own issues.  This is unavoidable, as the GUI needs to remain active while external access is running, and just something you'll need to learn how to handle.

Programmers are used to imperative programmer, where programs execute in a linear fashion and you always know how you got somewhere and the state of your application when you get there.  Reactive programming breaks this paradigm, and multi-threaded programming complicates it even more.  It's not really difficult to cope with, but it takes some time to get used to the new way of thinking.
