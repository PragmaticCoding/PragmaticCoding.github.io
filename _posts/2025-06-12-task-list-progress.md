---
title:  "Task Partial Results: Lists"
date:   2025-07-02 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/task-list-progress
PantsOnFire: /assets/images/PantsOnFire.png
ScreenSnap0: /assets/elements/TaskProgress4.png
ScreenSnap1: /assets/elements/TaskProgress5.png
ScreenSnap2: /assets/elements/TaskProgress6.png
ScreenSnap3: /assets/elements/TaskProgress7.png
ScreenSnap4: /assets/elements/TaskProgress8.png
ScreenSnap5: /assets/elements/TaskProgress9.png
ScreenSnap6: /assets/elements/TaskProgress10.png
ScreenSnap7: /assets/elements/TaskProgress11.png
ScreenSnap8: /assets/elements/TaskProgress12.png
ScreenSnap9: /assets/elements/TaskProgress13.png
ScreenSnap10: /assets/elements/TaskProgress14.png

Diagram: /assets/elements/ListProperties.png
TaskArticle: /javafx/elements/task-progress

JavaDocOL: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/ObservableList.html
JavaDocFXC: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/FXCollections.html

excerpt: "What if you want to monitor the progress of a Task that is collecting data into a List?  This article shows an approach to handling accumulated data in on ObserableList while a Task is running while still playing nicely with the FXAT."
---

# Introduction

In an earlier [article]({{page.TaskArticle}}) I looked at `Task` and how to track progress and partial results as a `Task` is running.  This can be a challenge, as any `Observable` value that can be linked to your layout can **only** be updated from the FXAT, while the action of the `Task` is designed to be run on a background thread.  The usual answer to this is to wrap any calls to update those `Observable` objects in a `Platform.runLater()` call.

In that earlier article, we looked at how the `Task.updateValue()` and `Task.updateMessage()` methods are designed to limit the number of `Platform.runlater()` calls that are executed, in order to avoid flooding the FXAT with jobs that would affect the performance of the GUI.  

We also looked at creating a custom `Property` in `Task` that could be updated in the same manner as `Task.value` and `Task.message` and then bound to elements in your layout.  Those methods use an internal `AtomicReference` field, that can safely be updated by multiple threads, to hold an temporary value that can later be transferred to the `Property` through `Platform.runLater()`.  

But `AtomicReference` relies on an "atomic" `getAndSet()` method to retrieve and update the value that it holds in a single, thread-safe, operation.  But what if you don't want to replace the value, but add to it?

This is the situaton that you get when your monitored results are in the form of an `ObservableList`.

## What the JavaDocs Say:
Let's have a look at the informational section at the top of the JavaDocs for `Task`:


> Suppose instead of updating a single value, you want to populate an ObservableList with results as they are obtained. One approach is to expose a new property on the Task which will represent the partial result. Then make sure to use Platform.runLater when adding new items to the partial result.
>
``` java
     public class PartialResultsTask extends Task<ObservableList<Rectangle>> {
         // Uses Java 7 diamond operator
         private ReadOnlyObjectWrapper<ObservableList<Rectangle>> partialResults =
                 new ReadOnlyObjectWrapper<>(this, "partialResults",
                         FXCollections.observableArrayList(new ArrayList<Rectangle>()));
   //
         public final ObservableList<Rectangle> getPartialResults() { return partialResults.get(); }
         public final ReadOnlyObjectProperty<ObservableList<Rectangle>> partialResultsProperty() {
             return partialResults.getReadOnlyProperty();
         }
    //              
         @Override protected ObservableList<Rectangle> call() throws Exception {
             updateMessage("Creating Rectangles...");
             for (int i=0; i<100; i++) {
                 if (isCancelled()) break;
                 final Rectangle r = new Rectangle(10, 10);
                 r.setX(10 * i);
                 Platform.runLater(new Runnable() {
                     @Override public void run() {
                         partialResults.get().add(r);
                     }
                 });
                 updateProgress(i, 100);
             }
             return partialResults.get();
         }
     }
```

They give us "one approach...", but no more.  This approach can work if the the updates to the list are relatively infrequent...

{% include notice_question type="primary" content = "How would you handle a situation where there are thousands of updates to the `ObservableList` from multiple threads that are coming milliseconds apart?" %}

There's no help from the JavaDocs for this situation.

# A Quick Review of the Task.updateMessage()

You really should go back and read that earlier [article]({{page.TaskArticle}}), but we'll have a quick look at the process for updating a single value.

`Task` has a number of `Properties` that are intended to be updated and changed from background threads, and it is instructive to see how `Task` internally handles these updates.  

`Task` has a field called `message` and the infrasture to update it from background threads.  The source code for `Task.updateMessage()` looks like this:

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
We have an internal, `private`, variable called `messageUpdate` which is an `AtomicReference`, and that will hold a transient value for updates to the `message Property`.  Because it is an `AtomicReference`, it is thread-safe, and can be updated freely without concurrency issues.  

The `updateMessage()` method, if run from the FXAT, will simply update `Task.message` directly.

When called from a background thread, however, it does something more complicated.  First, it uses `AtomiceReference.getAndSet()` to update `messageUpdate` and to check it's previous value.  If that previous value was `null`, then it knows that there wasn't any previous transient value waiting to be transferred to `Task.message` via `Platform.runLater()` and it needs to initiate that process.  So it creates a `Runnable` and schedules it on the FXAT via `Platform.runLater()`.

That `Runnable` does two things:

1. It transfers the current value of `Task.messageUpdate` to `Task.message`.
1. It sets the value of `Task.messageUpdate` to `null`.

If `Task.updateMessage()` is called from a background thread any time between `Platform.runLater()` being called and the `Runnable` being executed on the FXAT, then the previous value of `messageUpdate` will **not** be `null`, and it will not, therefore, need to launch a new `Runnable` on the FXAT.  In fact, the only thing that will happen will be that `Task.messageUpdate` has its value changed.  When the `Runnable` does execute on the FXAT, it will update `Task.message` with only that latest value that is in `Task.messageUpdate`.

In this way, `Task.updateMessage()` allows gazillions of updates from the background threads, but doesn't flood the FXAT with jobs.  As a matter of fact, you never have more than a single job pending on the FXAT at any given time for these updates. Transient values that never make it to `Task.message` would probably never have made it to the screen if `Platform.runlater()` had been invoked for every one of them in any case.  They certainly wouldn't have been seen by the user.

## About AtomicReference

`AtomicReference` is thread-safe because its "read" and "set" operations are conducted in a single, "atomic" operation.  This means that there's no possibility that the value could have been changed by some other thread accessing the same object in between the "read" stage and the "set" stage.

But this comes with a cost.

There is no way to *modify* a value in `AtomicReference`, we can only *replace* it.  Yes, there is a way to use the current value in an `AtomicReference` to calculate a new value for it, `AtomicReference.getAndUpdate()` but this will also *replace* the value.  So, we can do something like this:

``` kotlin
val atomicReference = AtomicReference<Int>(10)
val oldValue = atomicReference.getAndUpdate{it + 5}
```
This allows us to create a new value for `atomicReference` based on its old value, but doesn't simply modify the value already inside it.

This means that we cannot do this:
``` kotlin
val atomicReference = AtomicReference<MutableList<Int>>(mutableListOf())
val oldValue = atomicReference.getAndUpdate(it.add(5))
```
because the compiler will complain that `it.add(5)` returns `Boolean` when it was expecting `MutableList<Int>`.  This means that we will have to find some other way to deal with it.

# Simply Running the Updates in Platform.runLater()

Before we try anything else, let's see what happens if we take the approach from the JavaDocs, and just put every update on the FXAT via a call to `Platform.runLater()`.  Because, if this works, then we're all done!

I'm going to use the same type of application that I used in that earlier article, this will make it easier to follow if you did go back and read that article.


``` kotlin
class TaskProgress5App : Application() {
    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 800.0, 500.0)
        stage.title = "Task Progress Example"
        stage.show()
    }

    @OptIn(ExperimentalAtomicApi::class)
    private fun createContent(): Region = BorderPane().apply {
        val task = object : Task<Unit>() {

            val valueList: ObservableList<String> = FXCollections.observableArrayList()
            val waitingEvents = AtomicInt(0)

            init {
                updateMessage("")
                updateProgress(0, 100)
                setOnSucceeded { evt -> updateProgress(100, 100) }
            }

            private fun appendValue(var1: String) {
                if (Platform.isFxApplicationThread()) {
                    valueList.add(var1)
                } else {
                    println("Queuing: ${waitingEvents.incrementAndFetch()}")
                    Platform.runLater {
                        println("Performing: ${waitingEvents.decrementAndFetch()}")
                        valueList.add(var1)
                    }
                }
            }

            override fun call() {
                handleProcessing2(
                    { done, total -> updateProgress(done, total) },
                    { appendValue(it) },
                    { updateMessage(it) })
            }
        }
        center = VBox(10.0).apply {
            children += Label().apply {
                textProperty().bind(task.messageProperty())
            }
            children += ProgressBar().apply {
                progressProperty().bind(task.progressProperty())
                minWidth = 200.0
            }
            alignment = Pos.CENTER
        }
        right = TextArea().apply {
            textProperty().bind(ListToString(task.valueList))
        }
        bottom = HBox(Button("Run Task").apply {
            setOnAction { Thread(task).start() }
        }).apply {
            alignment = Pos.CENTER
            padding = Insets(20.0)
        }
    }
}

@OptIn(ExperimentalAtomicApi::class)
fun handleProcessing2(
    progressUpdater: (Long, Long) -> Unit,
    dataCollector: (String) -> Unit,
    messageUpdater: (String) -> Unit
) {
    val maxCount = 1000
    val threadCount = 50
    val job = Runnable {
        dataCollector("Thread ${Thread.currentThread().name} starting")
        for (x in 0..maxCount) {
            dataCollector("Thread ${Thread.currentThread().name}: $x")
            Thread.sleep(Duration.ofMillis(10))
        }
    }
    val sTime = LocalTime.now()

    progressUpdater(0, 100)
    messageUpdater("Initializing")
    Thread.sleep(Duration.ofMillis(1000))
    progressUpdater(-1, 100)
    messageUpdater("Processing")
    val threadPool = Executors.newFixedThreadPool(threadCount)
    repeat(threadCount) {
        threadPool.execute(job)
    }
    messageUpdater("ThreadPool Started")
    threadPool.shutdown()
    var counter = 0
    while (!threadPool.awaitTermination(4, TimeUnit.SECONDS))
        messageUpdater("Still Waiting ${counter++}")
    messageUpdater("Completed ${sTime.until(LocalTime.now(), ChronoUnit.MILLIS)}")
    progressUpdater(100, 100)
}

class ListToString(private val theList: ObservableList<String>) : StringBinding() {

    init {
        super.bind(theList)
    }

    override fun computeValue(): String? =
        theList.joinToString("\n")
}

fun main() {
    Application.launch(TaskProgress5App::class.java)
}
```
We have a `TextField` that has its `textProperty()` bound to `Task.valueList` through the `Binding` called, `ListToString`.  This means that as `Task.valueList` grows, the text in the `TextArea` should grow with it.

The background job uses an `ExecutorPool` of 50 threads to each run a simple job that just generates a `String` which is passed back to the `Task` and then sleeps for 10 ms.  When the `Task` gets each `String` from the background thread, it uses `Platform.runLater()` to add that `String` to `Task.valueList`.

This means that, with 50 threads each generating 1000 `Strings`, with a 10ms delay between them, we get 50,000 `Strings` submitted via 50,000 `Platform.runLater()` calls in a 10 second period.  This means it will generate 50,000 FXAT jobs in that 10s.

It looks like this when it is running:

![Screen Snap 1]({{page.ScreenSnap0}})

Honestly, when I first ran this without the `println()` statements, it just looked hung.  When I added them, I saw that `waitingEvents` rose up to 40K+ within a matter of seconds, the `Task` completed, and **then it continued for several minutes** to process all of the jobs on the FXAT and decrement the `waitingEvents` counter.

An earlier version of this code had the `ProgressIndicator` stay in Indeterminent mode through the processing.  It appeared to be stuck while the jobs were being executed.  So clearly this approach is not very different from running a long process on the FXAT as far as the GUI responsiveness is concerned.

{% include notice type="warning" content = "Never flood the FXAT with thousands and thousands of jobs submitted via `Platform.runLater()`" %}

When the jobs on the FXAT finally all complete, after 10 minutes or more, it looks like this:

![Screen Snap 2]({{page.ScreenSnap1}})

You can see that it took a little more than 11 seconds for the background threads to complete, which seems about right.  But the GUI was essentially frozen for the next 10+ minutes while the FXAT jobs were processed.

{% include notice type="info" content = "Clearly, the methodology suggested by the JavaDocs is not a viable approach." %}

# Using Synchronized Operations

The challenge is very well defined at this point.  We need to gather our updates together into batches such that we never have more than 1 or 2 jobs in the FXAT to update our `ObservableList`, but we cannot use the thread-safe `Atomic` classes to do this.  The answer, is to employ locks on our update `List` so that each thread, including the FXAT, can update it without causing data integrity issues.  

Let's look at how we'll do this....

The only thing that needs to change is our `Task`.  The inner workings of the update process are hidden from the rest of the code in the application, so it won't be impacted:

``` kotlin
val task = object : Task<Unit>() {

    private val dataList = mutableListOf<String>()
    val valueList: ObservableList<String> = FXCollections.observableArrayList()
    val counter = SimpleIntegerProperty(0)

    init {
        updateMessage("")
        updateProgress(0, 100)
        setOnSucceeded { evt -> updateProgress(100, 100) }
    }

    private fun appendValue(var1: String) {
        if (Platform.isFxApplicationThread()) {
            valueList.add(var1)
        } else {
            val sTime = LocalTime.now()
            synchronized(dataList) {
                if (dataList.isEmpty()) {
                    Platform.runLater {
                        val sTime = LocalTime.now()
                        synchronized(dataList) {
                            valueList.add(
                                "FX Event ${counter.value++} happened in ${
                                    sTime.until(LocalTime.now(), ChronoUnit.MILLIS)
                                }ms\n"
                            )
                            valueList.addAll(dataList)
                            dataList.clear()
                        }
                    }
                }
                dataList.add(var1 + " ${sTime.until(LocalTime.now(), ChronoUnit.MILLIS)}")
            }
        }
    }

    override fun call() {
        handleProcessing1(
            { done, total -> updateProgress(done, total) },
            { appendValue(it) },
            { updateMessage(it) })
    }
}
```
Here we've introduced a `private` field called `dataList` which is just a `MutableList<String>`.  This is what we are going to use to "batch" our updates to the `ObservableList` on the FXAT.  The rest of the changes are in `appendValue()`.

If you've looked at the source code for `Task.updateMessage()` above, then you should recognize the pattern here.  First, if we are on the `FXAT` then we just add the value to `valueList` without any fuss.  If we are *not* on the FXAT, then we need to add our new `String` to `dataList` and decide if we need to start a new job on the FXAT.  That determination is done by looking at `dataList` to see if it is empty before we add our new value.  If it is, then we need to start a new FXAT job.  

That FXAT job does three things:

1. Adds a message to `valueList` that says, "FXAT happened".  This way we can track how many updates end up in each update "batch".
1. Adds all of the items in `dataList` to `valueList`.
1. Clears out `dataList`.

All three things are done inside a `synchronized` block locked on `dataList`.

The last thing that `appendValue()` does is to add our new value to `dataList` after appending some timing information to it.

In order to avoid concurrency problems, all of this activity: checking if `dataList` is empty, possibly launching a new job on the FXAT and appending the new value to `dataList` is done in a `synchronized` block, locking on `dataList`.  
We should look at where this code runs:

* Every call to `appendValue()` runs on one of the 50 threads in the `ExecutorPool`.  There can, therefore, be multiple simultaneous access from many different threads to this code at any given time.
* The code that copies data from `dataList` to `valueList` and clears out `dataList` always runs from the FXAT, and may run at the same time that any number of threads are running `appendValue()`.

The `synchronized` blocks around every single access to `dataList` ensure that only one thread can execute code in either of those two blocks at any given time.

This means that when a background thread is adding a new value to `dataList`, no other background threads can do the same thing, and the `synchronized` block inside the job on the FXAT cannot run either.  When the job on the FXAT is running, no other background thread can enter the `synchronized` block in `appendValue()`.  If they cannot enter, they wait.

For this reason, the time calculation code doesn't just return `0` every time.  This information in the output will tell us how long any threads had to wait for other threads to complete their processing first.  There's a time calculation on both the background threads and the FXAT job.  We also count how many times it runs the FXAT job.

It looks like this:

![Screen Snap 3]({{page.ScreenSnap2}})

We can see that it now took about 15.5 seconds to complete the background task.  This means that the synchronization locking caused an approximate, cumulative 4 seconds of waiting time.  

On the other hand, the process finished almost immediately after the background tasks completed.  At no time did the GUI appear frozen or unresponsive, and everything seemed to operate smoothly.  

You can also see from the screensnap, that the FXAT job was invoked 299 times in that 15 seconds.  It never seemed to have to wait.

# About that Binding

In this example, the contents of the `TextArea` reflect the contents of `Task.valueList` through a custom `Binding`:

``` kotlin
class ListToString(private val theList: ObservableList<String>) : StringBinding() {

    init {
        super.bind(theList)
    }

    override fun computeValue(): String? =
        theList.joinToString("\n")
}
```
This might seem a little "heavy handed" at first.  The entire `String` is rebuilt every time that the `ObservableList` changes.  Surely it might be better to put a `ListChangeListener` on `valueList` instead, and then just apply the changes to the `TextArea`?

1. We really cannot make any assumptions about how the layout is going to use the `Properties` that we expose from our `Task`.  We need to design our implementation such that any "normal" usage of the `Property` is going to be functional.  That includes a "heavy-handed" `Binding` like this.

1. Remember that `String` in Java is immutable.  So any attempt to append data to a `String` is going to result in copying the entire `String` over and adding the new text.  Is this more efficient than `List.joinToString()`?  Maybe, but it's probably more a matter of degree than magnitude.


{% include notice type="info" content = "Bindings are recalculated in-line with the code that invalidates their dependencies." %}


`Bindings` are not executed through `EventHandlers` that are submitted as new jobs an the FXAT.  This means that you can measure their impact on performance directly.  In our FXAT `Runnable` if we move the code that generatest the "FX Event..." text from the top of the `synchronized` block to the bottom - after `valueList` has been modified, and *after* the `ListToString.computeValue()` has run - we get this:

![Screen Snap 4]({{page.ScreenSnap3}})

and this:

![Screen Snap 5]({{page.ScreenSnap4}})

You can see the top and the bottom of the timings, and as `valueList` gets bigger, the times get longer.  I checked and this is a progression through the output.  

I noticed that the first few elements added to each batch picked up measurable delays as the processing went on.  This was because the FXAT job started to take longer and longer to complete, so more background tasks (which completed very quickly) got backed up waiting for the FXAT job to release the lock.

I also noticed that the number of elements in the batches got larger as the processing went on.  Once again, the increased time for the FXAT job to release the lock meant that more background threads completed their `sleep(10)` while the lock was on, and therefore contributed to the batches.  Eventually, the wait became longer than the 10ms sleep time, so every batch included contributions from every thread.

{% include notice_question type="primary" content = "Is there anything we can do here to limit the time that the FXAT job spends in the syncronized block?" %}

Absolutely!  We need to extract the values from `dataList` and then clear it out inside the `synchonized` block, but we don't have to update `valueList` inside the synchronized block, since we can only ever update it from the single FXAT thread:

``` kotlin
private fun appendValue(var1: String) {
    if (Platform.isFxApplicationThread()) {
        valueList.add(var1)
    } else {
        val sTime = LocalTime.now()
        synchronized(dataList) {
            if (dataList.isEmpty()) {
                Platform.runLater {
                    val sTime = LocalTime.now()
                    val tempList = mutableListOf<String>()
                    synchronized(dataList) {
                        tempList.addAll(dataList)
                        dataList.clear()
                        valueList.add(
                            "FX Event ${counter.value++} step 1 in ${
                                sTime.until(LocalTime.now(), ChronoUnit.MILLIS)
                            }ms"
                        )
                    }
                    valueList.addAll(tempList)
                    valueList.add(
                        "FX Event ${counter.value++} done 1 in ${
                            sTime.until(LocalTime.now(), ChronoUnit.MILLIS)
                        }ms\n"
                    )
                }
            }
            dataList.add(var1 + " ${sTime.until(LocalTime.now(), ChronoUnit.MILLIS)}")
        }
    }
}
```
Here, I've added a `tempList` local variable which is just a `MutableList<String>`.  The elements from `dataList` are extracted into it, `dataList` is cleared and the lock is released.  Then, outside of the `synchronized` block, the contents of `tempList` are added to `valueList` and the `ListToString Binding` will run.  The output looks like this:

![Screen Snap 6]({{page.ScreenSnap5}})

You can see that we've stripped away almost 3 seconds from our background task runtime.  In fact, this is now only a little more than 1 second longer than the first version, without any locking at all.  I've added an intermediate timer, so you can see how long the `synchronized` part takes, and it's usually 0ms.  The `Binding` step seems to take longer now.  This may reflect a bigger continual load on my CPU from the background threads that possibly slows down the FXAT.

In my testing, I found that everything continued to work fine, even as I lowered the `sleep` duration in each thread down to one or two milliseconds.

# Returning a Value

What we've done so far is good, but it falls short of solving `Task` as an accumulator that delivers a final value.

The problem is that `Task.value` is private, and `Task.valueProperty` exposes `Task.value` as a `ReadOnlyObjectProperty`, even to sub-classes of `Task`.  The only way that we can update `Task.value` is by calling `Task.updateValue()`, and this uses the same `AtomicReference` logic as we saw in `Task.updateMessage()`, it won't work when we are trying to add items to a `List`.

It's one thing to add a custom `valueList` and expose that to the layout, but if we want our `Task` to work in a more generic fashion, then we need to have some way to transfer `valueList` to `Task.value` when the processing is complete.  So, this is the last item we need to address.

There's one issue that comes from using `Platform.runLater()` to batch update the `ObservableList`.  The `Task` is virtually guaranteed to complete before the all the jobs have been run on the FXAT.  

We saw this with the first version, just launching many, many jobs using `Platform.runLater()`.  The `Task` completed minutes before all of the jobs had been run on the FXAT, and any layout updates that would be triggered by the completion of `Task` would be incorrect.

Let's see how this impacts the layout.  To make things simpler, and easier to see where there are issues, we'll make the results of the `Task` the *size* of the `ObservableList` instead of the `List` itself:

``` kotlin
private fun createContent(): Region = BorderPane().apply {
    val task = object : Task<Int>() {

        private val dataList = mutableListOf<String>()
        val valueList: ListProperty<String> = SimpleListProperty(FXCollections.observableArrayList())
        val counter = SimpleIntegerProperty(0)
        val manualSize = SimpleIntegerProperty(0)

        init {
            updateMessage("")
            updateProgress(0, 100)
            setOnSucceeded { evt -> updateProgress(100, 100) }
        }

        private fun appendValue(var1: String) {
            if (Platform.isFxApplicationThread()) {
                valueList.add(var1)
            } else {
                val sTime = LocalTime.now()
                synchronized(dataList) {
                    if (dataList.isEmpty()) {
                        Platform.runLater {
                            val sTime = LocalTime.now()
                            val tempList = mutableListOf<String>()
                            synchronized(dataList) {
                                tempList.addAll(dataList)
                                dataList.clear()
                                valueList.add(
                                    "FX Event ${counter.value++} step 1 in ${
                                        sTime.until(LocalTime.now(), ChronoUnit.MILLIS)
                                    }ms"
                                )
                            }
                            valueList.addAll(tempList)
                            valueList.add(
                                "FX Event ${counter.value++} done 1 in ${
                                    sTime.until(LocalTime.now(), ChronoUnit.MILLIS)
                                }ms\n"
                            )
                        }
                    }
                    dataList.add(var1 + " ${sTime.until(LocalTime.now(), ChronoUnit.MILLIS)}")
                }
            }
        }

        override fun call(): Int {
            handleProcessing3(
                { done, total -> updateProgress(done, total) },
                { appendValue(it) },
                { updateMessage(it) })
            val tempSize = valueList.size
            Platform.runLater { manualSize.value = tempSize }
            return valueList.size
        }
    }
    center = VBox(10.0).apply {
        children += HBox(
            10.0,
            promptOf("Manual Size:"),
            Label().apply { textProperty().bind(task.manualSize.asString()) })
        children += HBox(
            10.0,
            promptOf("Counted Size:"),
            Label().apply { textProperty().bind(task.valueList.sizeProperty().asString()) })
        children += HBox(
            10.0,
            promptOf("Task Value:"),
            Label().apply { textProperty().bind(task.valueProperty().asString()) })
        children += HBox(
            10.0,
            promptOf("Task Completion:"),
            Label("None").apply {
                task.setOnSucceeded { evt -> this.text = task.get().toString() }
            })
        children += Label().apply {
            textProperty().bind(task.messageProperty())
        }
        children += ProgressBar().apply {
            progressProperty().bind(task.progressProperty())
            minWidth = 200.0
        }
        alignment = Pos.CENTER
    }
    right = TextArea().apply {
        textProperty().bind(ListToString(task.valueList))
    }
    bottom = HBox(Button("Run Task").apply {
        setOnAction { Thread(task).start() }
    }).apply {
        alignment = Pos.CENTER
        padding = Insets(20.0)
    }
}
```
In the layout, I've added a few `Labels` to display various values of the size of `Task.valueList`:  

* The one tagged with "Counted Size" is simply bound to the size of the `Task.valueList`.  I switched `valueList` over to a `ListProperty` instead of just an `ObservableList` because `ListProperty` has a `size Property`.
* The one tagged "Manual Size" is bound to a `IntegerProperty` that I added to `task` called `manualSize`.  The value is updated manually by `task.call()`.
* The one tagged with "Task Value" is bound to `task.valueProperty`.
* The one tagged "Task Completion" is manually updated through the `task.onSucceeded Property`, which calls `task.get()`.

All together, this is probably over-kill, but you'll see the problem.  

The type of `task` was changed from `Void` to `Int` so that we now have a value that we can grab.  Ordinarily, for something like this, you'd probably make the type of `task` an `ObservableList`, but keeping it as `Int` makes it easier to see the issues here.

Aside from the added and changed `Properties` in `task`, the only other change was in `task.call()`, which now looks like this:

``` kotlin
override fun call(): Int {
    handleProcessing3(
        { done, total -> updateProgress(done, total) },
        { appendValue(it) },
        { updateMessage(it) })
    val tempSize = valueList.size
    Platform.runLater { manualSize.value = tempSize }
    return valueList.size
}
```
As you can see, `task.call()` now returns `Int`.  Three lines have been added to the bottom of this method.  First we create a local variable called `tempSize` and initialize it with the size of `valueList`.  To update `task.manualSize` we need to be on the FXAT, so we'll need `Platform.runLater()` here.  Then, finally, we return valueList.size.

A few points:

* `Task.call()` itself runs in a background thread.
* We don't really know what invokes `Task.call()`.
* Whatever value goes into `Task.value` has to be updated on the FXAT.  This means that whatever calls `Task.call()` and receives an answer will need to invoke `Platform.runLater()` to update `Task.value`.  This implies some kind of delay between the end of `Task.call()` and the update of `Task.value()`.

Here's the results:

![Screen Snap 7]({{page.ScreenSnap6}})

There's the problem!

The actual, final size of `valueList` after all the FXAT jobs have completed is 50,502.  The size of `valueList` at the moment that `task.call()` completed was only 50,335.  The value that was captured by `Task.onSucceeded` and the value that went into `Task.value` was 50,335.  

I think you can see that if you weren't interested in the size, but actually wanted `Task` to return the `List` itself, you'd be short almost 200 entries.

## The Key Element of the Problem

The crux of the problem is that `Task` is designed such that `Task.call()` returns a value and that value is manually used to update `Task.value`.  However, `Task.call()` runs on a background thread, which means that updates to the accumulating return value, running on the FXAT, may not have completed at the time that `Task.call()` completes.  

{% include notice_question type="primary" content = "How do we ensure that `Task.call()` does not complete before all of the jobs running on the FXAT that are updating the results have completed?" %}

It turns out that the key factor is that `Task.call()` runs on a background thread.  This means that we have no problems running `Thread.sleep()` inside `Task.call()`.  All we need is a way to determine if all of the jobs on the FXAT have completed!

## Pausing at The End of Task.call()

Before we go there, let's have a look at how the `ThreadPool` is handled:

``` kotlin
val threadPool = Executors.newFixedThreadPool(50)
repeat(50) {
    threadPool.execute(job)
}
messageUpdater("ThreadPool Started")
threadPool.shutdown()
var counter = 0
while (!threadPool.awaitTermination(4, TimeUnit.SECONDS))
    messageUpdater("Still Waiting ${counter++}")
```
That `while{}` statement is important.  More correctly, the `ThreadPool.awaitTermination()` is very important.  This ensures that our method, running on the main background thread, doesn't return until after all of the jobs running on all of the threads have completed and the `ThreadPool` has shutdown.  

So, when we return from `handleProcessing3()`, we know that there are no more updates coming to `Task.dataList` from any background threads.  

This means that the **only** way that `Task.dataList` can change after we return from `handleProcessing3()` is from a job on the FXAT.  And we know that the only change that such a job makes to `Task.dataList` is to clear it out.  

Now we know how to determine when the last FXAT job has run, and the updates to `Task.valueList` are complete...when `Task.dataList` is empty!

And since we are still running on a background thread in `Task.call()`, we can wait for it!

``` kotlin
override fun call(): Int {
    handleProcessing3(
        { done, total -> updateProgress(done, total) },
        { appendValue(it) },
        { updateMessage(it) })
    while (!dataList.isEmpty()) {
        updateMessage("Waiting for completion: ${dataList.size}")
        println("Waiting for completion: ${dataList.size}")
        Thread.sleep(10)
    }
    Thread.sleep(100)
    updateMessage("All jobs completed")
    val tempSize = valueList.size
    Platform.runLater { manualSize.value = tempSize }
    return valueList.size
}
```
I've added a little `while{}` loop that uses `sleep(10)` to wait for the last FXAT job to run.  The first time I ran this I didn't have the last `sleep(100)` after the `while{}` loop and it still gave incomplete results.  

Remember how we used a temp variable to hold the contents of `dataList` so that we could clear out `dataList` and release the lock so that the background threads could start loading it up again without waiting for the `Binding.calculateValue()` to run?  Well that was causing a time difference between the clearing of `dataList` and the population of `valueList` on the FXAT, and we need to account for it here.

That last `Thread.sleep(100)` gives the FXAT enough time to complete the update of `valueList` after `dataList` has been cleared out.

This is what it looks like when it's done:

![Screen Snap 8]({{page.ScreenSnap7}})

As you can see, the last FXAT update takes 65ms to run, so the 100ms sleep is long enough to handle this.

And this is the console output:

```
> Task :run
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
Waiting for completion: 2319
```
You can see that the `while{}` loop with the `sleep()` ran about 20 times.  And now the counts are all the same, and correct.


# Conclusion

I think that the way that `Task` is designed to handle frequent updates from background threads is absolutely brilliant.  There a number of things to learn from this:

## Use Task!

Somehow, people seem to think that using `Task` is a heavyweight approach to background processing, and they prefer to just drop a `Runnable` into a new `Thread` and let it go.  

But `Task` is designed to provide the integration between your GUI and your background job, and to provide it in a way the behaves nicely with the FXAT. This is one place where the JavaDocs are dead-on:

> A fully observable implementation of a FutureTask. Task exposes additional state and observable properties useful for programming asynchronous tasks in JavaFX, as defined in the Worker interface.

So use it.  Use it every time!

## Look at the JavaFX Source Code

Sometimes, in the JavaDocs there are hints about interesting things that you simply cannot appreciate until you look "under-the-hood" and see how it is done.  Check out the JavaDocs for `Task.updateMessage()`:

> Updates the message property. Calls to updateMessage are coalesced and run later on the FX application thread, so calls to updateMessage, even from the FX Application thread, may not necessarily result in immediate updates to this property, and intermediate message values may be coalesced to save on event notifications.

I'm not sure that "coalesced" is the right word here, since it doesn't combine them together but simply replaces older, unprocessed, values with newer ones.  But this description strongly hints at something interesting going on behind the scenes.  

Looking at the source code for things like this shows you the *proper* way to handle complex issues with JavaFX.  Don't be scared to look, and most IDE's will take you right to the internal source code from the method call in your application code with a single click.

## Only Use `Platform.runLater()` Thoughtfully

I constantly see people online recommending, "Wrap it in `Platform.runLater()`".  I see this in tutorial articles as well as discussions and on StackOverflow.  

I consider this to be an anti-pattern, if not an out-and-out "code smell".  

Sure, in the solution presented here we did use `Platform.runLater()`, but in a considered and deliberate manner.  And also, it was wrapped inside a utility class and **not** integrated with any of the application/business logic at all.  

You may have noticed that code in `handleProcessing()` never changed throughout all of the examples in this article.  That's because it's logic was decoupled from the logic in the `Task`.  We updated `Task.call()` and `Task.appendValue()`, but never had to touch the business logic.  

At the other end, the GUI was connected to the `Task` through that `ListToString Binding`.  That `Binding` is owned by the layout, as are all other elements related to how the layout utilizes the results, and the partial results, from the `Task`.  If it turned out that the `ListToString` was causing performance issues with the GUI, then that's the layout's problem to solve.

Honestly, converting the `ObservableList` to a `String` to display it in a `TextArea` is a pretty bad design decision.  It would be much better to use a `ListView` which is designed to be used in this manner.  You could even change the output from `String` to some structured object, and then have better data available to format the display of the information.  

## Beware the JavaDocs

While the actual code that runs JavaFX is often brilliant, the associated JavaDocs often are sorely lacking.  In this particular example, the approach suggested by the JavaDocs is just bad - wrong even.  

What is also interesting is that the JavaDocs for `Platform.runLater()` contain this warning:

> NOTE: applications should avoid flooding JavaFX with too many pending Runnables. Otherwise, the application may become unresponsive. Applications are encouraged to batch up multiple operations into fewer runLater calls.

The approach suggested by the JavaDocs for `Task` totally disregard this warning.

I wish I could say this was a unique occurance, but lots of JavaDoc entries have dubious advice in their introductory sections.  

## Multi-threading is Always Complicated

In this application we had three kinds of threads:

1. The FX Application Thread (FXAT)
1. The main background thread to manage the processing
1. 50 `ExecutorPool` threads that did the actual work

It was extremely important to keep track of what activities take place on which thread.  For instance, the main background thread controlled the completion of the `Task.call()` logic.  It was extremely important that this thread not run on to completion before all of the `ExecutorPool` threads had finished.

These three thread types all shared access to one data element, `Task.dataList`.  Understanding how each thread used this data element and how access needed to be controlled to ensure that concurrency issues did not arise was key to architecting the solution properly.  We saw how keeping hold of the lock unnecessarily in the FXAT caused the entire process to take almost 3 seconds longer.

## This Solution is Still a Bit Kludgey

One thing that I do not like about this approach is that the intermediate results, while the `Task` is running, are accessed from the `Task` differently from how the final result is accessed.  

To be clear, you can still always access the final result from `Task.valueList`, but the rest of the `Task` infrastructure is designed around the result being in `Task.value`.  For instance, `Task.get()` delegates to `Task.value.get()`.  

But, unfortunately, there's no clean way to get the partial results into `Task.value` because it's private to `Task` and can only be updated through `Task.updateValue()`.  Yes, you could copy the entire `ObservableList` in `Task.valueList` over, but that doesn't seem to meet the definition of "clean" to me.
There doesn't seem to be any way to create a replacement for `Task` that accumulates rather than replaces, and still works the same way as `Task`.

You cannot achieve this by extending `Task` because key elements are `private`, and key methods are `final`.  You cannot simply use `Task` as a basis to create a brand new class that implements `FutureTask` and `Worker` because `Task` uses methods from JavaFX packages that aren't exported to outside modules.  This makes it very difficult to implement the `Event` firing mechanism in `Task`.

Maybe there is a better way to craft this into a generic solution that can be added to a library, but I can't find it.
