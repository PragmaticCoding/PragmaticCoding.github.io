---
title: "Part 13 - Feature Envy"
short_title: Part 13
permalink: /beginners/part13
screenshot_1: /assets/beginners/part10_1.png
excerpt: The last problem with our application is that it can save duplicate customer records, corrupting our database.  This means adding some rules to our database and telling our GUI when those rules have been broken.  We'll see how to handle exceptions from the back-end in our application.
---

# What You'll Learn

1. How easy it is to expand the application by a single field
1. Some more DRY
1. How to keeping code clean as we expand functionality

# Dealing With Our Ugly GUI and Our Ugly ViewBuilder Code

# Custom Widgets

When I build screens I tend to think about them in terms of smaller pieces as building blocks, and that these smaller pieces are often complete, self-contained pieces of functionality.  I think of them as "Custom Widgets".  I like the term "Widgets" better than "Nodes", because the term "Node" has a specific definition as a Class type in JavaFX.  "Widget", however, is a generic term commonly used in talking about GUI's and it simply refers to a screen element that interacts with the user.  So, a widget can be just about anything on the screen, but a `Node` refers to a JavaFX class.

A "Custom Widget" is something that somebody, probably you, has built, styled and configured.  It can be, but usually isn't, an extension of a `Node` class, and can be a container class with other widgets inside it.

Let's take a look at a piece code from our GUI:

``` java
private Node rowBox(String prompt, StringProperty boundProperty) {
    return new HBox(6, promptLabel(prompt), boundTextField(boundProperty));
}
```

The thing that this method returns is a custom widget.  It's an `HBox`, but it's been configured and populated with a `Label` and a `TextField`.  But it's not treated as an `HBox`.  It's really just a generic widget that performs a function and that is placed into a layout.  That's why it's returned as `Node`.

Now, what does this widget do?  Well...it puts a prompt `Label` on the screen and a `TextField` beside it.  The name "rowBox()" implies that it's going to be oriented horizontally - in a row.  

Think about these questions:  

Once we've instantiated it, do we care that it has a `Label` in it?  
: No, not really.  We care that there's something on the screen that indicates whatever our `prompt` parameter specifies, but we don't really care that it's a `Label`.  It could be a `Text`, or it could be an `ImageView`, or graphic of some sort.  It doesn't need to be a `Label`.  We don't care.

What about the TextField?
: We don't really care about that either.  It could be any kind of control that allows user input that can be bound to our `boundProperty` parameter.  Maybe `TextArea`, or some custom input control we downloaded in a library.  As long as we get user input, we don't really care.

The important point is that everything that `rowBox()` needs to do it's job is passed to `rowBox()` as a parameter.  When we get the result back from `rowBox()`, we don't care about any aspect of it except that we can put it in our layout.  It's self-contained.  We can treat it as a fairly anonymous custom widget, that does its job.

# Feature Envy

"Feature Envy" is when a component (usually a class) takes on functionality which doesn't really belong to it.  It grows more features so it can be more important!  

We want to avoid Feature Envy as much as possible.

Feature Envy is the opposite of the principle of, "Put Stuff Where it Belongs".  And this idea itself is really is just a natural outcome of following the Single Responsibility Principle, and DRY.  

Let's take a look at another method in our ViewBuilder to see how this works:

``` java
private Node boundTextField(StringProperty boundProperty) {
    TextField textField = new TextField();
    textField.textProperty().bindBidirectional(boundProperty);
    return textField;
}
```

We added this method to stick to DRY and to avoid repeating the code inside the method.  But let's take a closer look at it.

First of all, the return value is essentially another Custom Widget.  It's a bound `TextField`.

What does this code have to do with our layout, or our ViewBuilder class?  Nothing much other than the fact that it's called from it.  It doesn't use any fields from the class, nor does it call any other methods in the class.  In fact, it could be somewhere else and it wouldn't matter.  That's a problem, because stuff should only be in a class because it really needs to be there.

Let's look at it from the perspective of DRY.  In any normal business application, you're likely to have many screens, and each and every one of them is likely to have `TextFields` that are bidirectionally bound to `StringProperties`.  Are you going to *repeat* this method in every one of those ViewBuilders?  And is it going to be exactly the same code in each one?  Yes, it would be if you did it, and this would be a clear violation of DRY - on an application level.  

Now let's look at it from the perspective of the Single Responsibility Principle, but at the level of our ViewBuilder class.  It's only direct responsibility is to build and configure the View layout.  This method handles a detail outside of that responsibility, and should be delegated to somewhere else.  

Finally - and this is really telling - this method is only called from `rowBox()` which itself is another method that has nothing to do with our layout.  

# Let's Put These General Purpose Custom Widgets Somewhere Else

When something doesn't belong somewhere, then you have to put it someplace else.  Here we are talking about fairly generic methods, that accept plain data and standard JavaFX properties as parameters, and return configured standard JavaFX Nodes.  None of it has anything to do with our application directly.  So, "someplace else", means some other class in some other package.

I've created a new class with static methods called Widgets.  I'm putting it in a package called "widgets", and it looks like this:

``` java
public class Widgets {
    public static Node promptLabel(String contents) {
        return styledLabel(contents, "prompt-label");
    }

    public static Node headingLabel(String contents) {
        return styledLabel(contents, "heading-label");
    }

    public static Node styledLabel(String contents, String styleClass) {
        Label label = new Label(contents);
        label.getStyleClass().add(styleClass);
        return label;
    }

    public static Node promptedTextField(String prompt, StringProperty boundProperty) {
        return new HBox(6, promptLabel(prompt), boundTextField(boundProperty));
    }

    public static Node boundTextField(StringProperty boundProperty) {
        TextField textField = new TextField();
        textField.textProperty().bindBidirectional(boundProperty);
        return textField;
    }
}
```

It has all of the Feature Envy methods from our ViewBuilder class.  The name `rowBox()` has been changed to `promptedTextField()` because it needs to be more easily understood without any context.  Maybe `promptedInput()` would be better?  I don't know.  Naming things is hard!

All of this stuff is now out of our layout code:

``` java
public class CustomerViewBuilder implements Builder<Region> {

    private final CustomerModel model;
    private final Consumer<Runnable> saveHandler;

    public CustomerViewBuilder(CustomerModel model, Consumer<Runnable> saveHandler) {
        this.model = model;
        this.saveHandler = saveHandler;
    }

    @Override
    public Region build() {
        BorderPane results = new BorderPane();
        results.getStylesheets().add(Objects.requireNonNull(this.getClass().getResource("/css/customer.css")).toExternalForm());
        results.setTop(Widgets.headingLabel("Customer Information"));
        results.setCenter(createCentre());
        results.setBottom(createButtons());
        return results;
    }

    private Node createCentre() {
        VBox results = new VBox(6,
                Widgets.promptedTextField("Account #:", model.accountNumberProperty()),
                Widgets.promptedTextField("Name:", model.customerNameProperty()),
                Widgets.promptedTextField("eMail:", model.emailProperty()));
        results.setPadding(new Insets(20));
        return results;
    }

    private Node createButtons() {
        Button saveButton = new Button("Save");
        saveButton.disableProperty().bind(model.okToSaveProperty().not());
        saveButton.setOnAction(evt -> {
            saveButton.disableProperty().unbind();
            saveButton.setDisable(true);
            saveHandler.accept(() -> saveButton.disableProperty().bind(model.okToSaveProperty().not()));
        });
        HBox results = new HBox(10, saveButton);
        results.setAlignment(Pos.CENTER_RIGHT);
        return results;
    }
}
```

Once you take it out, you realize how much those Feature Envy methods were messing up the layout code.  You can take it in at a glance now.  In comparison, the previous version of our ViewBuilder was cluttered.

# One Last Point

Imagine for a minute that JavaFX came equipped with - straight out of the box - a Node class that did exactly what `promptedTextField()` does.  Let's call it `PromptedTextField`, and it had a constructor that took a `String` and a `StringProperty`.

Would you go and look at the source code for it to see how it works, or to check if it was working properly?

No, you wouldn't.  I might, but you wouldn't.  You'd use it and you'd be glad it was there and assume that it was going to work properly.

`Widgets` is really nothing different.  It's a set of utilities that are so generic that you'll use them over, and over, and over again.  And, because you use them all the time, you know that they will work properly.  Most of them, just like the ones here, are so simple that there's no reason to worry about them not working properly.

But they shift a huge burden of detail out of your layout code.  And the shift a huge burden out of your mind when you're working with your layout code.  That's a huge win.
