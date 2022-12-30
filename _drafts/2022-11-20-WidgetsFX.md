---
title:  "WidgetsFX: A Utility Libary to Simplify Layouts"
date:   2022-11-20 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/widgetsfx
ScreenSnap: /assets/posts/StarterFX.png
GitHubDialog: /assets/posts/StarterFX-GitHub.png
Diagram: /assets/posts/MVCI.png
excerpt: How do you start writing a JavaFX application?  Here's a self-contained project guaranteed to run out-of-the-box, with enough structure to get you started.
---

# Introduction

One of the things that I've heard people say about JavaFX goes something along the lines of, "FXML is structured and easier to understand than layout code in Java".

I'll acknowledge that dealing with FXML; reading it, understanding it, creating it and doing fancy stuff with it; is a skill that I've never taken the time to develop.  Some people have and... good for them.

But the other side of that, the idea that JavaFX layout in code are necessarily complex and difficult to understand is pure nonsense.  It's just that these people who are good with FXML haven't taken the to develop the techniques to write good layout code.

And I'd argue that good layout code is far, far easier to read, understand, maintain and create than FXML will ever be.  You just need to know how to do it.  

# Complexity in Layout Code

My experience is that complexity in JavaFX layout code comes from two sources:

1. Boilerplate, boilerplate and more boilerplate.
1. "Low-level" JavaFX components.

So let's take a look at these two things:

## Boilerplate Code

The boilerplate code in JavaFX is almost always configuration code. You'll instantiate a component, add some styling, load some data, maybe set up an EventHandler - stuff like that.

Most of the time, though, you're doing the same stuff over and over again - violating DRY every time. Also, if you consider layout and configuration as two separate responsibilities (which you probably should), then you're violating the "Single Responsibility Principle", too.

For me, I tend to use components in the same way, over and over. You find something that works, and you stick to it. For instance, I don't think you should ever put a Label on the screen without specifically styling it. I tend to have around 1/2 dozen kinds of styles that I'll use: one for data, one for prompts, one for screen titles, one for error messages, and a bunch of ones for different sizes of headings. The actual style definitions are determined by style sheets, but use cases pretty much stay the same - across applications.

So instead of:

``` java
Label label = new Label("This is a prompt");
label.getStyleClass().add("label-prompt");
HBox hBox = new HBox(6, label, somethingElse);
```

Over and over, I'll do this:

``` java
 .
 .
Label label = createPrompt("This is a prompt");
HBox hBox = new HBox(6, label, somethingElse);
 .
 .
 .
private Node createPrompt(String labelText) {
  Label label = new Label("This is a prompt");
  label.getStyleClass().add("label-prompt");
  return label;
}
```

But now I have no need to do anything with label except put it into a layout, so I don't even need the variable:

``` java
  .
  .
HBox hBox = new HBox(6, createPrompt("This is a prompt"), somethingElse);
  .
  .
private Node createPrompt(String labelText) {
  Label label = new Label("This is a prompt");
  label.getStyleClass().add("label-prompt");
  return label;
}
```

This is just an application of DRY.  We have the same two lines code repeated over and over, with only a single variation.  So we parameterize the difference and put the duplicate code into a method.  Now the layout code only contains the difference, tucked into the method call.

On top of that, and equally important, we now have a method with a meaningful name!  We have `createPrompt()`.

## Low-Level Components

The second issue, about the "low-level" components goes the same way. JavaFX gives you just about all of the basic building blocks you'll ever need, but different programmers are going to use them different ways.  Most programmers will establish different patterns of putting them together that they use all the time. So you can put those constructs into a class (if that makes sense), or into a utility library.

This really is an outgrowth of thinking about *all* of the parts of your layouts as custom components.  And this itself is an outgrowth of sticking to the "Single Responsibility Principle".

How does this work?

The idea is that you start at the top of your layout.  Let's say you're going to use a `BorderPane`.  If your sticking to the Single Responsibility Principle, then all you should really do in your top level method is to instantiate your BorderPane - and maybe do a little configuration.  To populate the parts of your `BorderPane` you need to delegate to other methods.  When something is delegated, the delegating method is no longer "responsible" for it.  So you'd end up with something like this:

``` java
private Region buildLayout() {
  BorderPane borderPane = new BorderPane();
  borderPane.setTop(createHeaderPanel());
  borderPane.setCenter(createDataEntryArea());
  borderPane.setBottom(createButtonBar());
  return borderPane;
}
```
This is pretty simple, which is why it's great.  Instantly we can tell that our layout is a `BorderPane`, which has the top, centre and bottom areas populated.  We can see exactly where to go if we want to know more about how any one of those areas are created.  

The way to think about this is that whatever is returned by `createHeaderPane()` and the other two methods are custom components.  So, `createHeaderPanel()` would return something called a `HeaderPanel`, whatever that is.  All we know is that it's a subclass of `Node` so that it can be put into a layout.  And when we look into `createHeaderPanel()`, we might find that it too is divided up into different custom components, each of which is defined in its own method.

# Extending These Patterns

Eventually, you come to the second screen in your application and you find that you are using the same patterns again.  If you put another `createPrompt()` method in you next screen, you'll be violating DRY - but this time at an application level.  

The obvious course is to refactor `createPrompt()` out into into a utility class as a static method.  Let's create a class called `Labels`.  It's going to hold just customized `Labels`, so the name make sense.  Since it's a whole class just for creating `Labels`, we don't need the "create" prefix for the method, so we can just call it `Labels.prompt()`.

But let's say that our second screen has a `Label` formatted up to be an error.  It's still just styling, so we can extend our `Labels` class like this:

``` java
  public class Labels {
    public static Label styledLabel(String labelText, String styleClass) {
      Label results = new Label(labelText);
      results.getStyleClass().add(styleClass);
    }

    public static Label prompt(String labelText) {
      return styledLabel(labelText, "label-prompt");
    }

    public static Label error(String labelText) {
      return styledLabel(labelText, "label-error");
    }

  }
```
This is a bit of a two-for.  Now we have the generic `Labels.styledLabel()` which gives us a single call to instantiate a styled `Label` - and that we added so that we wouldn't violate DRY inside our `Labels` class.  Along with that we have the descriptive `Labels.prompt()` and `Labels.error()`.

## A Super Important Point

While all we've done here is to just keep applying DRY to our application as it grows, there's something else really important that's happened.

As soon as we added the second screen that used `createPrompt()`, the code in that method no longer had anything specific to do with either layout.  As such, it no longer belonged in the layout code.

In other words, the only code that you have in a layout class should be code that uniquely applies to that layout.

If you follow this process, and adhere to this rule, then anybody who ever looks at any of your layout code will never be burdened with understanding code that isn't specific to that layout.

**It is shocking how small your layout code becomes when you follow this process.**

# Custom Components

I mentioned earlier that all of those builder methods, like `createHeaderPanel()` in the `BorderPane` example should be thought of as returning custom components.  In this way, your entire layout is composed of custom components.  `Labels.prompt()` returns a custom component.  It's a pretty simple custom component, just some styling an a standard `Label`, but it's still best to think of it as a custom component.

But if you follow DRY, you're going to find that you're creating some very similar custom components over and over.  This is normal.  

JavaFX gives you just about all of the basic building blocks you'll ever need, but different programmers are going to use them different ways.  Most programmers will establish different patterns of putting them together that they use all the time. So you can put those constructs into a class (if that makes sense), or into a utility library.

## Example: PromptFieldButton

For instance, a pattern I've used a fair amount is an HBox with a Label styled as a prompt, a TextField for user input and a Button. The Button is configured so that it's the "Default" when the content of the TextField is non-empty. The Button is disabled immediately on clicking, and then re-enabled when its action is completed.

When you get into the details of it, there's a fair bit of configuring and fussy stuff to work out.  TextFields and Buttons are taller than Labels (usually), so you probably have to change the HBox alignment from `Pos.TOP_LEFT` to `Pos.CENTER_LEFT` to make it look right.  Things like that.

KMgFYiB8MYJM



Really though, when it comes to specific implementations you only need to supply 4 things: the prompt text, a property to bind the TextField to, a Button label, and an action handler for the Button (I would use a Consumer<Runnable>). You can supply all of these in a single method call, so once again, you can drop the results right into a layout without a variable reference. Even better, put the call and the Consumer declaration into a method and give it a name:

``` java
private Node accountLookupBox() {
   Consumer<Runnable> actionHandler = afterAction -> {
       doSomething();
       afterAction.run();
  }
  return Library.promptFieldButton("Account", model.AccountProperty(), "Search", actionHandler);    
}
```

Once again, do you care how `Library.promptFieldButton()` works when you are looking at the layout? Probably not.

One of the reasons that it's easy to find it is that there's shockingly little code left when you've stripped out all the boilerplate configuration code and the repeated patterns. The ONLY stuff that's left is very basic layout, and very custom configuration.

On top of that everything has names that tell you what they do. Labels.prompt() is pretty clear, as is Labels.data(), or Labels.h1(). Library.promptFieldButton() is clear once you know what it does, but if you're looking at the GUI, and the code at the same time, it's no mystery. Inside the layout, methods like accountLookupBox(), which could easily just be one line if you in-line that consumer, keep the main layout code clean and give a meaningful name to part of the layout.
