---
title:  "Dealing With Events"
date:   2024-03-16 00:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/dispatch_chain
Diagram: /assets/posts/MVCI.png
Dispatch1: /assets/elements/EventDispatchChain1.png
Dispatch2: /assets/elements/EventDispatchChain2.png
DemoRunning: /assets/posts/MvciDemoRunning.png
DemoDone: /assets/posts/MvciDemoDone.png
Chain1: /assets/elements/Chain1.png
Chain2: /assets/elements/Chain2.png
excerpt: Advanced concepts in Event handling.  Dealing with the Event Dispatch Chain.
---

# Introduction


# EventHandlers Describe Actions Triggered by Events

`Events` are the "messages" that are passed around your JavaFX application.  At any given time there could be hundreds or even thousands of `Events` active in your JavaFX application.

It's easy to get the impression that an `EventHandler` is "triggered" by the JavaFX object where it is defined, but this isn't technically correct.  That JavaFX object will "fire" an `Event` and that `Event` is passed around through the JavaFX components of the application according to some specific rules.  In most cases, it will land back at the JavaFX object that fired the `Event` (also known as the "source") and if it has an `EventHandler` of the correct type defined on it, that `EventHandler` will be invoked.

This may sound like I'm splitting hairs, but this can be an important concept to understand.  JavaFX objects will often be passed `Events` that they did not trigger.  JavaFX objects can, therefore, act on `Events` that they did not trigger.  JavaFX objects can also intercept `Events` that they did not trigger and prevent them from reaching the JavaFX object that fired the `Event`.

## Event Dispatch Chain

There's technically one thing wrong with the description above:  The "source" isn't really the key element.  It's the "target".  Every `Event` that's fired has a target assigned.  Typically, this is the `Node` that you'd view as having "fired" the `Event`, so it borders on being just an issue of semantics.  However, if you look at any official documentation for this stuff, it will talk about "target", not "source".  On top of that "source" is still an actual thing, and part of the information stored in the `Event`.

Let's look at how `Events` are passed around the JavaFX application, it's something called the, "Event Dispatch Chain".  For this, I'm going to brazenly scoop some images from the Oracle tutorial, because they are good.

Here's our application window:

![Dispatch Chain Window]({{page.Dispatch1}})

In case it's not clear, we have a window with a `Rectangle` at the top, and below it a `Pane` holding a `Circle` and a `Triangle`.  

Here's the hierarchy of our `SceneGraph`:

![Dispatch Hierarchy]({{page.Dispatch2}})

This is what you'd expect:  There's `Stage` holding a `Scene` that has a `Group` which, in turn holds a `Rectangle` and a `Pane`.  The `Pane` holds the `Circle` and the `Triangle`.

For any given target `Node` in that hierarchy, the event dispatch chain will be the shortest route from the `Stage` to that `Node`.  It's that simple.

There are two phases to event routing, the "Event Capturing Phase" and the "Event Bubbling Phase".  In the Event Capturing Phase, the `Event` is passed to every `Node`, in order, starting with the `Stage` and ending with the target.  If the target in our diagram above was the `Triangle`, then the `Event` would be passed to the `Stage`, then the `Scene`, then the `Group` followed by the `Pane` and then the `Triangle`.  In this phase, the `Event` is handled by any `Handler` attached to the `Node` for that `EventType` as a `Filter`.

Bubbling up is the same process in reverse, in this case from the `Triangle` to the `Stage` and the `Events` are handled by any `Handler` attached to the `Node` for that `EventType` as a `Handler`.

Any of these `Handlers` attached to any of the `Nodes` in the Dispatch Chain, either as a `Filter` or a `Handler` can call `Event.consume()` to "consume" the `Event`.  Once the `Event` has been consumed, it won't be delivered to any more `Nodes`.  In this way, a `Node` can prevent other `Nodes` from seeing, and responding to an `Event`.

Viewed from this perspective, it's much easier to see `Events` as messages that are passed around the `Nodes` in your application.

# Event Dispatch Chain Example

This is reproduced from an earlier article:

## An Example

``` java
public class ChainedEvents extends Application {

    public static void main(String[] args) {
        launch(args);
    }

    private final StringProperty text = new SimpleStringProperty("");

    @Override
    public void start(Stage primaryStage) {
        Scene scene = new Scene(createScene());
        primaryStage.setScene(scene);
        scene.addEventFilter(Event.ANY, evt -> logHandler("Filter", "Scene", evt));
        scene.addEventHandler(Event.ANY, evt -> logHandler("Handler", "Scene", evt));
        primaryStage.addEventFilter(Event.ANY, evt -> logHandler("Filter", "Stage", evt));
        primaryStage.addEventHandler(Event.ANY, evt -> logHandler("Handler", "Stage", evt));
        primaryStage.show();
    }

    private Region createScene() {
        BorderPane results = new BorderPane();
        addHandlers(results, "BorderPane");
        TextArea textArea = new TextArea();
        textArea.setStyle("-fx-font-family: 'monospaced';");
        textArea.textProperty().bind(text);
        textArea.setMinWidth(700);
        results.setRight(textArea);
        Button button = new Button("Click Me!");
        addHandlers(button, "Button");
        StackPane stackPane = new StackPane(button);
        addHandlers(stackPane, "StackPane");
        results.setCenter(stackPane);
        results.setMinHeight(800);
        return results;
    }

    private void addHandlers(Node node, String name) {
        node.addEventFilter(Event.ANY, evt -> logHandler("Filter", name, evt));
        node.addEventHandler(Event.ANY, evt -> logHandler("Handler", name, evt));
    }

    private void logHandler(String handlerType, String name, Event event) {
        if (event.getEventType().equals(MouseEvent.MOUSE_MOVED))
            return;
        if (event.getEventType().equals(MouseEvent.MOUSE_ENTERED_TARGET))
            return;
        if (event.getEventType().equals(MouseEvent.MOUSE_EXITED_TARGET))
            return;
        String time = LocalTime.now().format(DateTimeFormatter.ofPattern("hh:mm:ss:SSS"));
        String className = event.getTarget().getClass().getSimpleName();
        String formatted = String.format("%-13s %-15s %-20s %-10s %-15s", time, name, event.getEventType(), handlerType, className);
        System.out.println(formatted);
        text.set(formatted + "\n" + text.get());
    }

}
```
Which looks like this, when it runs and button is clicked:

![Showing Filters and Handlers]({{page.Chain1}})

This is what's going on here:

We have a `Stage` which holds a `Scene` which holds a `BorderPane` which holds a `StackPane` which holds a `Button`.  That's the chain of containers from the top down to the `ActionEvent` source.  There are `EventFilters` and `EventHandlers` on every one of those objects, so that we can see how the `Events` are propagated and handled.  The output in the `TextArea` is in reverse chronological order.

You should notice:

* There's a lot of `Events`, not just the `Button ActionEvent`.
* The code to remove the mouse movement `Events` was necessary to prevent there being just too much output.
* At 5:27:58 the chain for the click on the `Button` starts, with `MOUSE_PRESSED` caught by the `Stage` filter.
* The `Button ActionEvent` starts when the mouse button is released.
* The `MOUSE_RELEASED` handler on the `Button` seems to happen after the `ActionEvent` has been handled by all of the objects.
* There are three mouse based `Events`: `MOUSE_PRESSED`, `MOUSE_RELEASED` and `MOUSE_CLICKED`.  They appear to be independent from the `Button's` `ActionEvent`.
* All of mouse based `Events` appear to be consumed by the `EventHandler` on the `Button`, since they don't bubble up any further.
* The mouse based `Events` all have a target of `LabeledText`.  This is because I clicked right on the middle of the `Button`, on the words "Click Me!".  `Button` is just a `Region` and it contains `Nodes` inside it to hold the contents.  The mouse clicks targeted these inner `Nodes` of `Button`.
* Look at the `MOUSE_ENTERED Events`.  The mouse enters all of the Nodes from `Scene` to `Button` in one movement, and it seems to handle each of those separately, without passing through a successively longer chain for each source, which makes sense.

## Event Consumption

The last twist is that any one of those `EventFilters` or `EventHandlers` on any of those `Nodes` can invoke the `consume()` method on the `Event` and swallow it up.  This prevents any other `Nodes` further along the "Capture" and "Bubbling" phases from seeing the `Event`.

I took the code above and changed this line:

``` java
scene.addEventFilter(Event.ANY, evt -> logHandler("Filter", "Scene", evt));
```
to:
``` java
scene.addEventFilter(Event.ANY, evt -> {
            logHandler("Filter", "Scene", evt);
            if (evt.getEventType().equals(ActionEvent.ACTION)) {
                evt.consume();
            }
        });
```
This will cause the `Scene` to consume the `ActionEvent` in the "Event Capturing Phase", so that the `Event` never gets to the `Button` (or anything contained in the `Stage`).

The output now looks like this:

![Consuming an Event]({{page.Chain2}})

Which is what you would expect.  All of the `Events` propagate through all of the `Nodes` except the `ActionEvent` from the `Button` which is seen by just the `Stage`, and then the `Scene`, which consumes it.
