---
title:  "Demystifying PseudoClasses in JavaFX"
date:   2022-01-24 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/pseudo_classes
excerpt: Pseudo Classes are the best way to handle on/off state changes in a Node in JavaFX.  But it's very badly explained in the JavaDocs and hard to understand.  This article should clear that up.
image1: /assets/elements/PseudoClass1.png
image2: /assets/elements/PseudoClass2.png
image3: /assets/elements/PseudoClass3.png

---

## What's a PseudoClass?

Have you ever noticed how a `Button` changes a little when you hover your mouse over it?  Or darkens to look like it's pressed down when you hold down a mouse button in it?  These presentation changes are usually accomplished through the use of `PseudoClasses`.

`PseudoClasses` are specialized CSS selectors which represent on/off states of some value of a `Node`.  One or more aspects of the styling of a `Node` will be changed when the `PseudoClass` is turned on, and then returned back to their original state when the `PseudoClass` is turned off again.  

They're useful because the styling is all handled for you behind the scenes.  The normal use case is to tie the `PseudoClass` state to a `Boolean Property`, and then bind that `Property` to a `Property` in the Model or some other part of the GUI.  Sometimes the `PseudoClass` state is control by `EventHandlers`.

### Built-In PseudoClasses

There are a number of predefined `PseudoClasses` in JavaFX that you can use without any special programming.  For instance, many controls have a `disabled` pseudo class which is automatically maintained for you whenever the control's `Disabled` property changes.  

The standard Modena CSS has definitions like this:

``` css
.spinner:disabled {
    -fx-opacity: 0.4;
}
```

You can override these in your own stylesheets to make the control behave differently when it's disabled.

### Create Your Own PseudoClass

The built-in `PseudoClasses` are useful, but sometimes you need to create your own `PseudoClass` to handle state changes in ways unique to your application.  For instance, you might want to change the colour of a `Label` if the data it displays indicates an error condition.

This article is about how to create a brand new `PseudoClass` and how to link it into your screen `Node` so that you can style it via your CSS.

## The JavaDocs for Pseudoclass are Horrible

If you want to look, you can find them [here](https://openjfx.io/javadoc/16/javafx.graphics/javafx/css/PseudoClass.html).

The introduction at the top reads like this:

> PseudoClass represents one unique pseudo-class state. Introducing a pseudo-class into a JavaFX class only requires that the method Node.pseudoClassStateChanged(javafx.css.PseudoClass, boolean) be called when the pseudo-class state changes. Typically, the pseudoClassStateChanged method is called from the protected void invalidated() method of one of the property base classes in the javafx.beans.property package.<br><br>Note that if a node has a default pseudo-class state, a horizontal orientation for example, pseudoClassStateChanged should be called from the constructor to set the initial state.<br><br>
The following example would allow "xyzzy" to be used as a pseudo-class in a CSS selector.

Yikes!  That doesn't really explain anything unless you already understand it.

And then they've included the following example code:

``` java
  public boolean isMagic() {
       return magic.get();
   }

   public BooleanProperty magicProperty() {
       return magic;
   }

   public BooleanProperty magic =
       new BooleanPropertyBase(false) {

       @Override protected void invalidated() {
           pseudoClassStateChanged(MAGIC_PSEUDO_CLASS, get());
       }

       @Override public Object getBean() {
           return MyControl.this;
       }

       @Override public String getName() {
           return "magic";
       }
   }

   private static final PseudoClass
       MAGIC_PSEUDO_CLASS = PseudoClass.getPseudoClass("xyzzy");

```

Seriously, I don't think this is a great way to explain how it works.  They've even called it `magicProperty`, which might be an admission that they understood they weren't explaining it very well when they wrote it.

Not to mention that `MyControl.this` is never defined anywhere.  Also, there's a typo (which I've fixed here) in the call to `pseudoClassStateChanged()`.  That doesn't help either.

## How Does it Really Work?

In reality, `PseudoClass` is a lot easier to understand than you might think.  There's no magic, and it's really easy to use `PseudoClass`.

### Creating the PseudoClass

In the example code from the JavaDocs, the first part that's really important is the single line:

``` java
private static final PseudoClass
    MAGIC_PSEUDO_CLASS = PseudoClass.getPseudoClass("xyzzy");
```

What does this do?

It does two things, one you can see, and one that you can't.

The first is to create an instance of `PseudoClass` as a static field.  Does it need to be static?  Technically, no, but as we'll see in a little bit, it's probably a good idea to implement it this way.

The second thing it does, which you cannot see, is that it updates a static `Map` deep inside `PseudoClass`, adding whatever name you used to initialize your `PseudoClass` - in this case, "xyzzy".  The `Map` makes sure that only one element with the same name exists in your application.  

If you try to instantiate another `PseudoClass` anywhere in your application with the same name, it will just reuse the `Map` entry it already has for that name.  That's not really a big deal, but you should avoid using names that are already defined in JavaFX, like "hover" or "disabled", because there might already be code that's going to be manipulating any `PseudoClasses` with that name.

### Setting the PseudoClass Value

The only other part of the example code you need to understand is this:

``` java
@Override protected void invalidated() {
    pseudoClassStateChanged(MAGIC_PSEUDO_CLASS, get());
}
```

Even this is too complicated.  You could use something like this:

``` java
    pseudoClassStateChanged(MAGIC_PSEUDO_CLASS, true);
```

What does this do?  `pseudoClassStateChanged()` is a method in `Node`.  You pass it a `PseudoClass`, and a new value (true or false) for that `PseudoClass` in that Node.  Under the hood, this method then adds or removes this `PseudoClass` to its list of active `PseudoClasses`.  In reality, this list is really just a list of the names that were given to the `PseudoClasses` when they were instantiated.

### Important Things to Know About PseudoClass

PseudoClass isn't part of the Node
: The `PseudoClass` itself isn't installed into the `Node`, formally declared as part of it, or prepped in the `Node` in any special way at all.  You can instantiate a `PseudoClass` anywhere and pass it to any `Node`'s `pseudoClassStateChanged()` method, and it will work.  Furthermore, you can call a `Node's` `pseudoClassStateChanged()` from any number of places using completely different instances of a `PseudoClass` with the same name, and it will work exactly the same as if you had only used one instance of the same `PseudoClass` repeatedly.


Node.pseudoClassStateChanged() is a public method.  
: This means that there's no need to extend any `Node` just to implement a `PseudoClass` on it.  You can manage the `PseudoClasses` on a `Node` from outside the `Node`.  This is absolutely not clear from the JavaDoc example.

The mechanism connecting PseudoClass to CSS isn't our concern
: How the `Node` actually turns the list of active `PseudoClasses` into styling changes an your screen is something that we simply don't need to know.  As long as your CSS has the selectors defined, it will work.

PseudoClass itself isn't Reactive
: It's important to note that `pseudoClassStateChanged()` itself has **NOTHING** to do with Properties or Bindings.  It's a plain vanilla Java method.  

PseudoClass instances aren't functionally unique
: There is nothing particularly special or specific about any instance of `PseudoClass` other than its name and the way that names are managed.  The following code is perfectly fine:
``` java
    PseudoClass xyzzy = PseudoClass.getPseudoClass("xyzzy");
    myButton.pseudoClassStateChanged(xyzzy, true);
````
It's probably a bit inefficient if it's going to be called a lot, just because there's a fair bit of code running inside of `PseudoClass.getPseudoClass()`.  So it might be best to declare it as a static field inside of whatever class holds this code.

## Connecting PseudoClasses to Properties and Bindings

So we've seen that `PseudoClasses` are just normal Java and are implemented through normal imperative programming techniques.  But if you want to build a reactive JavaFX application, then you need to control the `PseudoClasses` through `Properties` and `Bindings`.

How do you do that?

The way to connect imperative code to `Properties` is through `Invalidation Listeners`.  This is exactly what the example from the JavaDocs does, although it does it in the most complicated way possible.  

Essentially, what an `Invalidation Listener` does is to define a block of code that will run whenever the system decides that the value held in a property is no longer valid because it's potentially been changed, or some other properties that it's been bound to have changed.  Calling `get()` on the `Property` will cause it to recalculate it's value, but you can also run whatever other code you want in the `Listener`.  This is where you can call `Node.pseudoClassStateChanged()`.

The Property that you're listening to can be anything, anywhere.  It doesn't need to be a field in the `Node`, and it doesn't need to be a `BooleanProperty` either.  The `Listener` just needs to be defined in some scope that has access to both the `Node` and the `Property`.  The `Listener` would then contain whatever logic you need to turn a change in your `Property` value into a boolean value to be sent to `Node.pseudoClassStateChanged()`.

### A Simple Example

In this example, we're going to look at the "normal" use case for PseudoClass, a BooleanProperty which is part of the Model and which is connected to a PseudoClass through an `InvalidationListener`:

``` java
public class PseudoClassDemo extends Application {

    private BooleanProperty displayInRed = new SimpleBooleanProperty(false);

    @Override
    public void start(Stage stage) throws IOException {
        Scene scene = new Scene(createContent(), 320, 240);
        scene.getStylesheets().add(PseudoClassDemo.class.getResource("test.css").toString());
        stage.setScene(scene);
        stage.show();
    }

    private Region createContent() {
        PseudoClass redDisplay = PseudoClass.getPseudoClass("red");
        Label label = new Label("This is the label text");
        displayInRed.addListener(inv -> {
            label.pseudoClassStateChanged(redDisplay, displayInRed.get());
        });
        Button button = new Button("Click Me");
        button.setOnAction(evt -> displayInRed.set(!displayInRed.get()));
        return new VBox(20, label, button);
    }

    public static void main(String[] args) {
        launch();
    }
}
```
Here's the CSS file for this:
``` css
.label:red {
   -fx-text-fill: red;
}
```
When it runs, it looks like this:

![Before Click]({{page.image1}})

After the `Button` is clicked, it looks like this:

![After Click]({{page.image2}})

I think this strips virtually all of the mystery out of `PseudoClass` creation.  It is possible to cut the `BooleanProperty` completely out of this if you want and call `label.pseudoClassStateChanged()` directly from the `Button` action - although this is not a good idea in terms of creating a Reactive JavaFX program:

``` java
public class PseudoClassDemo2 extends Application {

    private boolean showInRed = false;

    @Override
    public void start(Stage stage) throws IOException {
        Scene scene = new Scene(createContent(), 320, 240);
        scene.getStylesheets().add(PseudoClassDemo2.class.getResource("test.css").toString());
        stage.setScene(scene);
        stage.show();
    }

    private Region createContent() {
        PseudoClass redDisplay = PseudoClass.getPseudoClass("red");
        Label label = new Label("This is the label text");
        Button button = new Button("Click Me");
        button.setOnAction(evt -> {
            showInRed = !showInRed;
            label.pseudoClassStateChanged(redDisplay, showInRed);
        });
        return new VBox(20, label, button);
    }

    public static void main(String[] args) {
        launch();
    }
}
```
Once again, this isn't a good practice.  It's included here just to show how `PseudoClass`  is just regular Java code.

### A More Complicated Example

At this point, we're not really talking about `PseudoClass` itself any more, but about strategies around transforming non-boolean `Properties` into CSS pseudo-class changes.  

Let's look at how to turn a `StringProperty` into pseudo-class states:

``` java
public class PseudoClassDemo3 extends Application {

    private final StringProperty alertLevel = new SimpleStringProperty("Normal");

    @Override
    public void start(Stage stage) throws IOException {
        Scene scene = new Scene(createContent(), 300, 100);
        scene.getStylesheets().add(PseudoClassDemo3.class.getResource("test.css").toString());
        stage.setScene(scene);
        stage.show();
    }

    private Region createContent() {
        PseudoClass redDisplay = PseudoClass.getPseudoClass("red");
        PseudoClass orangeDisplay = PseudoClass.getPseudoClass("orange");
        Label label = new Label("This is the label text");
        alertLevel.addListener(inv -> {
            label.pseudoClassStateChanged(redDisplay, false);
            label.pseudoClassStateChanged(orangeDisplay, false);
            switch (alertLevel.get()) {
                case "Warning" -> label.pseudoClassStateChanged(orangeDisplay, true);
                case "Emergency" -> label.pseudoClassStateChanged(redDisplay, true);
            }
        });
        return new VBox(20, label, createButtons());
    }

    private Node createButtons() {
        Button warningButton = new Button("Warning");
        warningButton.setOnAction(evt -> alertLevel.set("Warning"));
        Button emergencyButton = new Button("Emergency");
        emergencyButton.setOnAction(evt -> alertLevel.set("Emergency"));
        Button normalButton = new Button("All Clear");
        normalButton.setOnAction(evt -> alertLevel.set("All Clear"));
        return  new HBox(10, normalButton, warningButton, emergencyButton);
    }

    public static void main(String[] args) {
        launch();
    }
}
```

And the CSS file:

``` css
.label:red {
   -fx-text-fill: red;
}

.label:orange {
   -fx-text-fill: darkorange;
}
```

Which looks like this when the "Warning" button is clicked:

![Warning]({{page.image3}})

The `Buttons` just update the `StringProperty`, putting a new value into it.  The `StringProperty` has an `InvalidationListener` added to it, in a scope that has access to the `Label` so that the `InvalidationListener` can call its `pseudoClassStateChanged()` method.  

The `InvalidationListener` just turns off all of the custom `PseudoClasses` in the `Label`, and then uses a `switch` statement to turn on a specific `PseudoClass` depending on the value in the `StringProperty`.

## Conclusion

One good thing about the way that `PseudoClass` is introduced in the JavaDocs is that it reaffirms that the designers of JavaFX intended it to be used as a Reactive framework.  You can tell this because they take care to explain it in the context of a `BooleanProperty` embedded into a `Node`.

To be perfectly fair, though, the second sentence in the JavaDocs introduction is a complete and concise summary of everything in this tutorial:

>Introducing a pseudo-class into a JavaFX class only requires that the method Node.pseudoClassStateChanged(javafx.css.PseudoClass, boolean) be called when the pseudo-class state changes.

However, everything else in the JavaDocs makes it very difficult to believe it could be that simple.  Hopefully this tutorial has cleared that up.
