---
title:  "Should You Use FXML?"
date:   2025-03-10 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/fxml-or-not
PantsOnFire: /assets/images/PantsOnFire.png
ScreenSnap0: /assets/elements/TableColumn0.png
ScreenSnap1: /assets/elements/TableColumn1.png
ScreenSnap2: /assets/elements/TableColumn2.png
ScreenSnap3: /assets/elements/TableColumn3.png
ScreenSnap4: /assets/elements/TableColumn4.png
ScreenSnap5: /assets/elements/TableColumn5.png
ScreenSnap6: /assets/elements/TableColumn6.png
ScreenSnap7: /assets/elements/TableColumn7.png
ScreenSnap8: /assets/elements/TableColumn8.png
ScreenSnap9: /assets/elements/TableColumn9.png
ScreenSnap10: /assets/elements/TableColumn10.png

Diagram: /assets/elements/ListProperties.png
part1: /javafx/elements/observable-classes-generics
part2: /javafx/elements/observable-classes-typed
beginners: /beginners/intro

excerpt: FXML.  There are all kinds of claims about its value, but are they right?
---

# Introduction

From time to time the question about, "To FXML or not to FXML" comes up on Reddit and other social media, and I'm know as being firmly in the "Not FXML" camp.  I've had a few people message me about this, so I thought it might be a good time to examine this topic in depth.

Full disclosure:  
: I haven't used SceneBuilder and FXML since about 2015, so my only experience with it was as a beginner to JavaFX and I have no intention of ever using it again.  While I'm not an expert user of FXML, I am, however, very much aware of most of its capabilities and quirks because I read about them all the time.  I've also seen tons of FXML when looking at other programmers' projects on GitHub, and helping people with issues on-line.  I wouldn't hesitate to say that I know more about FXML and SceneBuilder than the average JavaFX beginner or intermediate programmer.  I just don't use it.

# True or False - Claims About FXML

There are a lot of claims about all of the wonderous benefits of using FXML.  Are they true?  Let's take a look at them and see...

## Separation of Concerns

This is the claim you hear the most, from the Oracle tutorial for FXML we have this:

> FXML is an XML-based language that provides the structure for building a user interface separate from the application logic of your code. This separation of the presentation and application logic is attractive to web developers

There's actually a lot to unpack in these two sentences.  With FXML, we have two components, the FXML file and the FXML Controller, so any talk of "separation" has to speak to the relationship between these two elements.  So let's concentrate on that...

When we talk about "Separation of Concerns", we are really talking about "coupling", or rather, keeping low levels of coupling.  And what this means is that you can make changes to one element of your application without having to make changes to another.  Does FXML achieve this?

Specifically, for FXML, we are concerned about whether or not we can make changes to the FXML file without needing to make changes to the FXML Controller, and visa versa.  What do these two components "know" about each other?

Most FXML implementation that I see follow a basic pattern:

* EventHandler subroutines in the FXML Controller are exposed to the FXML file.
* Named Nodes are exposed to the FXML Controller through @FXML annotations.

This is pretty easy.  The FXML file needs to be able to invoke subroutines in the FXML Controller in order to actually do anything.  So at a minimum, the FXML file is coupled to this aspect of the FXML Controller.  You can make any changes that you like to the FXML Controller, as long as you don't make critical changes to those subroutines which would impact the FXML file.

The other direction is tighter.  If you want to get data in or out of any Node on the screen, you'll need a *typed* reference to it declared as a field in your FXML Controller.  This means that everything that isn't purely static content in your GUI will need to be exposed to the FXML Controller.  You *have* to have each one of those `Nodes` in your FXML file, and they have to be the correct type of `Node` or your Controller code will break.  This also creates a dependency the other way,  your FXML file now requires that the FXML Controller has some code that loads data into these `Nodes`.

Is there some way to remove all this coupling?  Obviously, no... But can we do anything to make it "one way" coupling?  

It turns out that we can.  If we expose all of the elements that might generate `Events` that we want to handle in the FXML Controller to that Controller by giving them id's in the FXML file, then we can configure them, including attaching `EventHandlers` to them, in the FXML Controller.  If you do this then you don't have any references from the FXML file to the FXML Controller.

I haven't looked at enough FXML systems to say definitively that this technique is *never* used, but... I've never seen it used.

What about in the other direction? Can we create an FXML Controller that doesn't need any references to `Nodes` defined in the FXML file?

This turns out to be more complicated.  It is possible to define data elements as `Properties` in your FXML Controller that are exposed to the FXML file, and can be used in bindings defined in the FXML file.  The downside to this is that the FXML file is now even more tightly coupled to the FXML Controller, as it needs to know about all of the data elements available in the FXML Controller.  

Except that this won't work.  

FXML does not support bi-directional binding.  This is something that apparently has been a "top priority" for over a decade, and it's still not been implemented.  Without bi-directional binding, you cannot update those `Properties` in the FXML Controller and you cannot get data out of your GUI.  So this is a dead end.  

Finally, we haven't even considered cases where you have dynamic layouts, or layouts that are dependent on data.  These simply cannot be defined purely in FXML and will need code in the FXML Controller that performs this layout.

Now, let's go back to "Separation of Concerns".  What separation do we have?

In some cases, we have separation of *most* of the details of the actual layout, and the styling and configuration.  For instance, the FXML file might be the only component that knows that a couple of `Labels` are displayed in a `VBox` or an `HBox`, but if those `Labels` have dynamic content, then the FXML Controller needs to know that they are `Labels` and what data needs to be stuffed into them.  

I highly doubt that this is the level of "separation" that programmers are looking for when they expect "Separation of Concerns".  

But that Oracle tutorial doesn't even seem to be talking about this, it says:

> provides the structure for building a user interface separate from the application logic of your code

Application logic?  This leads to the next claim about the benefits of FXML...

## FXML Supports MVC

Also from that Oracle tutorial for FXML:

> From a Model View Controller (MVC) perspective, the FXML file that contains the description of the user interface is the view.

In MVC, the View isn't just a layout, it's a complete, independent user interface.  That means it handles everything GUI related.  This includes data binding and handling GUI events like resizing, mouse dragging and button clicks.  When one of these events requires activity outside of the GUI, then it communicates with the Controller to initiate that activity.  

You should understand that there are `Events` and there are "Actions".  `Events` are very specifically JavaFX GUI elements, and in FXML they are specified in the FXML file, and implemented in the FXML Controller (or they can be implemented and defined in the FXML Controller).  "Actions" are a much more generic concept and aren't necessarily GUI elements at all.   

I know that this sounds like nit-picking, but it's an important practical consideration when talking about MVC.  Let's look at what typically happens when you click a `Button` that requires some business logic to be run...

1. The first thing that happens is that the `Button` is disabled because you don't want to be able to initiate a the action again before it completes.  There may be other GUI set-up things you want to do, like clearing out some data from some `Nodes`, starting up a `ProgressBar` or putting up a "Please Wait" message.
1. The next thing that happens is the action is run, often in a background thread.  
1. Once the action completes, the Presentation Model is updated as required.
1. Finally, the `Button` is re-enabled, the `ProgressBar` is removed, and the "Please Wait" message is cleared.

The first and last steps are 100% GUI activity and should belong solely to the View.  Steps 2 and 3 are actions that should be taken by the Controller and the Model.  However, if you consider FXML file alone to be the View, it has no way to actually perform steps 1 and 4.  To implement steps 1 and 4, you'll have to write code in the FXML Controller!

What does this really mean?

As a result of the coupling - that cannot be removed - between the FXML file and the FXML Controller, these two elements have to be considered together as a single unit that comprises the MVC View!  Not the MVC View and Controller.  

Once you look at it this way, everything else makes much more sense and you don't have the cognitive dissonance that results from treating the FXML and FXML Controller as View and MVC Controller.  

But you don't automatically get an MVC Controller, you'll have to deliberately create one.  We'll look at this later.

This is actually the most dangerous of all of the false claims about FXML.  That's because, if you buy into this then you'll actually architect your system incorrectly

## Multiple Layouts With One Controller

The idea that the layout is a self-contained element, separate from the GUI logic, suggests that it is possible to create a single FXML Controller that works with multiple FXML layouts.  Also, that it's possible to do the reverse, and create multiple FXML Controllers that all work with the same FXML layout file.  Perhaps this second scenario is easier to picture.  You could have a single layout for a CRUD type application, and have a different FXML Controller for "create", "update", "read" and "delete".  

In practice, I'm not sure that anyone does this, or that it would be a good idea.  Essentially, you'd be coupling all of those FXML Controllers to each other through the FXML file.  Meaning that if you needed to change the FXML file to accommodate some change to one of the FXML Controllers, it could impact all of the other FXML Controllers.  The same thing happens in the reverse implementation where all of the FXML files are coupled through their shared FXML Controller.

My sense is that trying to do this makes your application more complicated, not simpler.

### But the Layout is Separate From the Logic!

It's important to recognize that "physically separate" and "uncoupled" are **NOT** the same thing.  It should be clear that although the FXML file and the the FXML Controller are two distinctly separate things, they are, in fact, very, very, very tightly coupled.  You literally cannot add any non-static element to your FXML file without making corresponding changes to the associated FXML Controller.

So what is the advantage to having the layout separate from the logic when both are still tightly coupled?

I'm not sure that there is one.  

I guess you could argue that you can concentrate on the layout without worrying about the GUI logic.  But is that actually an advantage?  Personally, I prefer the exact opposite, and I'll usually add one element at time to my layout while flushing out the support for it in the Model, Controller and Interactor as I go.  In this way, my GUI is always fully operational in so far as the elements that have been implemented and any issues that arise from downstream concerns are addressed immediately.  And that's going a lot further than just building the GUI logic as I go.

## FXML`s Declarative Syntax

This is touted as a great advantage to FXML, that it has a declarative syntax with a hierarchical structure which reflects the structure of your scene graph.

I *think* that this means that it is supposed to be easy to look at an FXML file and understand what it's doing, and to quickly find the elements that you are interested in and to understand how they are configured and relate to each other.

Let's see if this is true...

While I would prefer not to include a huge FXML file, I think we need to look at one to understand this issue.  So here's the FXML file from [Trinity on GitHub](https://github.com/trinity-xai/Trinity/blob/v2024.12.13/src/main/resources/edu/jhuapl/trinity/fxml/ManifoldControl.fxml):


All right, maybe if you're an FXML expert user you can look at these 462 lines of FXML and make some sense out of it at a glance...but I can't.  

I don't really know anything about the project this came from.  I don't know if it would be considered "good" FXML or not.  I don't know if the author was an expert or a beginner.  My sense is that it is fairly typical, and is probably what came out of SceneBuilder.

What I can say for sure is that it's monolithic.  SceneBuilder/FXML tends to push programmers in this direction, because I see this a lot.  It is possible to carve a layout up into smaller parts, but the FXML management then becomes a headache all in itself.  So it's rarely done.

This layout has **everything** encased in an `AnchorPane` that has only one child, a `TabPane`, but it has no anchoring specified.  Once again, this is something the SceneBuilder seems to encourage because you see it a lot.  Like, really...a lot.  The `AnchorPane` appears to have no purpose in the layout, but you have to scroll through the entire file to see that it only has the `TabPane` in it.  You also have to scroll through the whole file to see that the `TabPane` has 4 `Tabs` defined.

Let's look at the first `GridPane` defined.  It has this block of `RowConstraints`

``` xml
<rowConstraints>
    <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
    <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
    <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
    <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
    <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
    <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
    <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
    <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
    <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
    <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
    <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
    <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
    <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
    <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
</rowConstraints>
```
Yeah, you have to count them.  There's 15 and they are all the same.  You cannot do DRY with FXML.  Need to change the height of all of them?  Edit 15 lines (or do it 15 times in SceneBuilder).  Needless to say, you wouldn't do anything this crude in code.  And there are 26 more `GridPane` rows scattered throughout this file that use the same parameters!

Moving on to the content of the `GridPane`, we'll just look at a small section:

``` xml
<Label text="Number of Components"/>
<Spinner fx:id="numComponentsSpinner" editable="true" prefWidth="100.0" GridPane.rowIndex="1"/>
<Label text="Number of Epochs" GridPane.rowIndex="2"/>
<Spinner fx:id="numEpochsSpinner" editable="true" prefWidth="100.0" GridPane.rowIndex="3"/>
<Label text="Nearest Neighbors" GridPane.rowIndex="4"/>
<Label text="Negative Sample Rate" GridPane.rowIndex="6"/>
<Label text="Local Connectivity" GridPane.rowIndex="8"/>
<Spinner fx:id="nearestNeighborsSpinner" editable="true" prefWidth="100.0" GridPane.rowIndex="5"/>
<Spinner fx:id="negativeSampleRateSpinner" editable="true" prefWidth="100.0" GridPane.rowIndex="7"/>
<Spinner fx:id="localConnectivitySpinner" editable="true" prefWidth="100.0" GridPane.rowIndex="9"/>
```
First off, I guess we have to assume that the first `Label` goes in row/colum 0/0, because it has nothing specified.  I'm guessing this is what SceneBuilder does by default???  

The `Spinner` that goes with that first `Label` is actually declared in the next line, as well as the next `Label`/`Spinner` combination.  After that we get the next three `Labels` and then we find that the `Spinners` that go along with them follow after the last `Label`.  The rest of the `GridPane` seems to go more like that.  Perhaps this is simply the order in which the contents were added in SceneBuilder??  

I wonder what's in all these `Spinners`?  Integers?  Decimals?  What are there ranges?  How do they increment, and what are the default values?  Do they change dynamically?  You can't tell any of this without looking at the FXML Controller, because clearly that's where this stuff is defined.  I don't know if you can define these parameters in SceneBuilder, but the author has chosen not to here.  This is probably fairly common.  BTW:  The associated FXML Controller in this project is nearly 800 lines long.  

What can see here from these 462 lines of FXML is that the assertion that the "declarative, hierarchical syntax" of FXML is somehow easy to read, understand and maintain is just not true.  

I think it's clear that well structured and organized layout code is infinitely easier to read, understand and maintain than the equivalent FXML.

# What Are the Actual Benefits of FXML?

As far as I can tell, there is only one:  You can use SceneBuilder.  

It's up to you to decide if SceneBuilder is something that you find value in.  It's entirely possible that the drag-and-drop approach of SceneBuilder makes it easier for you to visualize how your layouts will work as you construct them.  

A lot of people mention that they find the ability to quickly run a simulation of the layout in SceneBuilder to be valuable.  For me, I find it just as easy to run the code and see it actually running.  While I imagine that SceneBuilder is probably a bit faster, I don't find that incremental builds in Gradle take very long at all, and I have never hesitated to launch a test run because I didn't want to spend time waiting.

I have worked on large systems where the specific screens I am developing are deep down a workflow that a significant amount of time to navigate to in the actual application.  I those cases, I wouldn't develop a screen layout "in situ", I'd whip up a mini-application for testing the layout (and the Model, Controller and Interactor for that screen).  

The downside of ScreenBuilder is that it strongly encourages you to develop the layout in isolation.  In reality, the layout needs to work hand-in-glove with the Presentation Model.  I like to let the GUI dictate the needs of the Presentation Model, and create the relationships between the `Nodes` in the layout and the `Properties` in the Presentation Model as I build the layout itself.  In this way, I don't just create the layout, I create a good part of the entire MVCI construct that it's part of at the same time.

## SceneBuilder as a Beginners' Tool

One of the biggest problems that I have with JavaFX is that it is a huge, complicated library with a nearly complete lack of good tutorial material.  This is an absolute nightmare for beginners.  

There is this idea that SceneBuilder, as a graphical, drag-and-drop interface for JavaFX GUI design, somehow makes JavaFX more accessible to beginners.  I seem to remember that Oracle touted SceneBuilder as huge benefit to switching over from Swing in the early days.

But for a beginner, starting up SceneBuilder is probably like sitting in the cockpit of a modern jet fighter when you've never even flown a kite before.  There are a dizzying array of `Controls` you can use, and a head-spinning number of options that you can fiddle with for each one.  And there's no explanations for any of it (at least there weren't years ago).

My main rememberance of SceneBuilder is the countless hours that I frittered away trying to make something...anything work the way I wanted it to.  I was lucky enough that I was on the clock, and I quickly realized that I couldn't keep throwing this time away when I was supposed to be productive.  So we decided to abandon SceneBuilder after a few months.  Things got better after that.

I often see comments from beginners that indicate that they are having the same experience.  I've also seen the the resultant FXML that beginners create with SceneBuilder, which are often full of silly things like wrapping the whole layout in an `AnchorPane` with a single child and no anchors.

### Dealing With the Complexity of FXML

Beginners, by definition, don't understand what they are doing and there's always a very big element of "paint by numbers" in their work.  This is especially true because the SceneBuilder -> FXMLLoader -> FXML Controller process seems to add some layer of inpenetrable magic that they cannot get past.  It's clear that most beginners don't even understand that the output of `FXMLLoader.load()` is just a `Node` subclass of some sort.  They only ever see it referred to as `root` and dropped immediately into a `Scene`, so they think that's all you can do with it.

Once you realize that FXML itself doesn't actually give you anything of value, you can easily see it instead as the *cost* of using SceneBuilder.  

Over the decades, I've used all kinds of different screen generators in different systems, I've even written one myself for a "green screen" system.  They all have to include some method to allow programmers to handle the stuff that doesn't neatly fall into values and parameters that can be easily included in the screen building tool.  Some use "hooks".  Some provide a place to enter code and subroutines.  Some just generate code that you can modify yourself - I think Swing had one of those.

SceneBuilder has FXML and the FXML Controller, along with the FXMLLoader that ties them together.  That's how it deals with things that you cannot easily define in SceneBuilder.  It introduces a layer of complexity that you do not have with coded layouts.

Let's face it.  Complexity is a huge issue with JavaFX, even without FXML.  

Think about some beginner who scratches his head for hours trying to figure out why that `Node` where he's set `visible` to `false` is still taking up space in his layout.  Why would they think they would need to set `managed` to `false` as well?  How would they even know what `managed` meant?  How are they supposed to cope with the magic that happens inside `FXMLLoader`?

It's clear that beginners have difficulty understanding how elements inside an FXML Controller relate to the associated layout.  It's also clear that they have trouble understanding how to apply JavaFX concepts to an FXML/FXML Controller context, and that they even have trouble applying basic Java concepts to that context.  

# The Cost of FXML

The first cost of FXML, as you can see from that giant FXML example above, is that it actually makes it more difficult to read and understand how a layout works than the equivalent in well constructed and organized code.

But the most important cost of FXML is that it makes it very difficult, if not impossible, to develop a library of helpers, builders and factory methods (or the equivalent in classes) that simplify and standardize layout creation the way that *you* approach it.

Programmer complaints about JavaFX seem to revolve around one of two things:  Too much boilerplate, or the lack of some particular `Node` or `Control` class.  

What do people mean by "boilerplate" with regards to JavaFX?  

Usually, it centres around the idea that you have to instantiate a `Node` as a variable, and then call several of its methods to configure it.  Then put the `Node` into the layout.  Something like this:

``` java
Label label = new Label();
label.getStyleClass().add("some-selector");
label.textProperty().bind(model.someProperty());
HBox hBox = new HBox(10, label, otherNode);
```
If you have ten `Labels` in your layout, and they are all configured in a similar way, then you've got 30 lines of boilerplate, which bloats your layout code and makes it difficult to read.

But you don't have to live with this.  You can create a static builder method that will do this for you:

``` java
public class Labels {

   public static Label promptLabel(ObseravbleStringProperty boundProperty) {
      Label label = Label();
      label.textProperty().bind(boundProperty);
      label.getStyleClass().add("some-selector");
      return label;
   }
}
```
And you can call it like this:

``` java
HBox hBox = new HBox(10, Labels.promptLabel(model.someProperty()),.....);
```
And there you go, no more boilerplate.  You can do this locally, as a method in your ViewBuilder class, or you can assemble a library of these methods that you put into a separate project.  

You *can* do some of this stuff with FXML.  You obviously cannot use builders, but you can create custom classes that compile into a jar file that you import into SceneBuilder to do some of the same things.

Which leads to the second point, missing `Node` classes...

There are many instances in JavaFX where the only tools that we are given are very generic.  Take text entry as an example.  We have `TextField`.

Now, `TextField` by itself is just a box that you can type anything into.  But `TextField` supports the inclusion of a `TextFormatter` which allows you to take complete control over the input into a `TextField`.  You can create `TextFormatters` for money amounts, integer amounts, postal codes, or phone numbers, or anything else you can think of.  Personally, I wouldn't use `TextField` without a `TextFormatter` for anything other than truly free-form text entry.

Let's look at phone numbers which have different formats all over the world.  In Canada and the US we use something like "#(###)###-####".  In France they use ""## # ## ## ## ##", while in Ukraine they use "### ## ### ####", and in China it's, "## ### #### ####".  And in many of these places, if not all, the format is different for calling in-country.  

One possible implementation is to have a different `TextFormatter` for every phone number layout.  `TextFormatter` is a `Property` of `TextField`, so it is possible to change the `TextFormatter` on-the-fly, or bind it to some other value.  This suggests some possible implementation approaches.  Alternatively, you could create builder methods to return specific styles of phone number entry.  For instance:
I feel that seeing one can make my point
``` kotlin
children += HBox(10.0, promptLabel("Phone Number:"), frenchPhoneNumberTF(phoneNumber)))
```
And now none of the mechanics of setting up the `TextField` are exposed to the layout code at all.  Alternatively,

``` kotlin
children += HBox(10.0, promptLabel("Phone Number:"), phoneNumberTF(countryCode, phoneNumber)))
```
works too.

If you were in a domain where you really did need a range of different formats based on the country code, you'd probably be tempted to have a `ComboBox` for the country selection, and then a `TextField` for the phone number.  You see this on websites all the time.  In this case, I'd probably create a custom control that had both the `ComboBox` and the `TextField` in it, along with their associated prompt Labels, and the connections between the country code and the `TextFormatter` could be established internally.  You could include a `Property<Orientation>` which would determine if the elements were layed out horizontally or vertically.

You'd use it like this:

``` kotlin
children += PhoneNumberEntryBox(countryCode, phoneNumber, orientation)))
```
Can you do this with FXML?

Somewhat.  You cannot access the builder approach, but you could create `FrenchPhoneNumberTF` class and use it.  You can reference `PhoneNumberEntryBox`, but you cannot connect the `countryCode` or `phoneNumber` parameters via FXML.  Nor can you access the `orientation` parameter in FXML.  In truth, you could make `orientation` a `StlyeableProperty` and have a custom attribute for it in a CSS file, which might be just as easy.  

But here's the thing:  Anyone that spends a lot of time writing JavaFX applications is going to encounter the same use cases, that require the same customized or custom `Controls`, over and over again.  Integrating them into your layouts is trivial - trivial to the point where they might as well be native `Nodes` in the JavaFX library - when you code your layouts by hand.  It's much more difficult to do the same with FXML.

## The Importance of Builders and Custom Controls

I simply cannot over-emphasize the importance and value of this.  

The core concept is that there is layout, and there is configuration, and there is logic.  

Configuration is where the boilerplate and repeated code lives.  Simply refuse to repeat yourself - apply DRY contstantly and consistently - and you'll find yourself extracting the configuration out of your layouts.  This leaves the layouts to be...layout.

This is the really big issue (in my opinion) with FXML.  It splits the logic out from layout, but leaves much of the configuration in the layout.  And the rest of the configuration gets mingled in with the logic.  

And the truth is that almost all of the configuration is a small set of paramaters that are always combined in the same way for each use case.  And the use cases are limited in number.

In the real layout code that I write, you'll almost never see me use something like, `new Label()`.  That's because I'm almost always going to configure that `Label` in one of four or five ways, and I have builders for each of them.  I do the same thing for many other `Controls`, too.  I have custom `TableColumns` that have `TableCells` for a number of data types like dates, money, integers and booleans.  

When you look at layout code, you don't care about the details of how a `Label` created through a call to `Labels.dataLabel(model.someProperty)` is configured.  You know it's a `Label` and it's configured to display data.  You also know that `Labels.dataLabel()` has been used hundreds of times, is thoroughly tested, and works properly.  So it's not likely to be a source of any problem you're having.  

Because of this, it's not just code that's not cluttering up your layouts, it's not cluttering up your mind either.  Do you worry about how `new Label("Something")` works?  No, of course not.  Nor do you worry about `Labels.dataLabel()`.

# FXML With a Framework

If you are going to develop real applications that actually do something, and you want to do it right, you'll need to adopt one of the commonly used frameworks like MVC, MVVM or my own MVCI.  These frameworks all isolate your UI logic from your business (or application) logic, and provide the "separation of concerns" that make systems easier to maintain and enhance.  

We now know that FXML doesn't give you a head start on MVC, because the FXML Controller is not an MVC Controller.  So here we'll look at how you would go about implementing a framework in both an FXML and non-FXML application, and see how they are different...

{% include notice_kotlin %}

If I go into Intellij Idea and use the "New -> Project..." wizard to create a JavaFX project, I'll get a "Hello World" application that uses FXML.  So let's do that and see what it looks like:

``` kotlin
class HelloApplication : Application() {
    override fun start(stage: Stage) {
        val fxmlLoader = FXMLLoader(HelloApplication::class.java.getResource("hello-view.fxml"))
        val scene = Scene(fxmlLoader.load(), 320.0, 240.0)
        stage.title = "Hello!"
        stage.scene = scene
        stage.show()
    }
}

fun main() {
    Application.launch(HelloApplication::class.java)
}
```
and the FXML Controller looks like this:
``` kotlin
class HelloController {
    @FXML
    private lateinit var welcomeText: Label

    @FXML
    private fun onHelloButtonClick() {
        welcomeText.text = "Welcome to JavaFX Application!"
    }
}
```
while the FXML file looks like this:

``` xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.Insets?>
<?import javafx.scene.control.Label?>
<?import javafx.scene.layout.VBox?>

<?import javafx.scene.control.Button?>
<VBox alignment="CENTER" spacing="20.0" xmlns:fx="http://javafx.com/fxml"
      fx:controller="ca.pragmaticcoding.fxmlstuff.HelloController">
    <padding>
        <Insets bottom="20.0" left="20.0" right="20.0" top="20.0"/>
    </padding>

    <Label fx:id="welcomeText"/>
    <Button text="Hello!" onAction="#onHelloButtonClick"/>
</VBox>
```
This is good enough for an example.  

Now, let's look at what the exact same application would look like without FXML:

``` kotlin
class HelloNoFXML : Application() {
    override fun start(stage: Stage) {
        with(stage) {
            title = "Hello!"
            scene = Scene(createContent(), 320.0, 240.0)
            show()
        }
    }

    private fun createContent(): Region = VBox(20.0).apply {
        padding = Insets(20.0)
        alignment = Pos.CENTER
        val welcomeText = Label()
        children += welcomeText
        children += Button("Hello ").apply {
            onAction = EventHandler { welcomeText.text = "Welcome to JavaFX Application!" }
        }
    }
}

fun main() {
    Application.launch(HelloNoFXML::class.java)
}
```
And you can count and see that both versions have 20 lines of Kotlin code, but the FXML version also has 8 lines of layout FXML.

In my opinion, the non-FXML version has the advantage that it's incredibly staight-forward.  You can look at it and you can see that `createContent()` is a builder, and that everything in it is just plain, vanilla Kotlin code.  You can see instantly how the action of the `Button` relates directly to the `Label` and how they are both in a `VBox` with a specific alignment, padding and spacing.  

But now, let's imagine that we are going to something more involved with this.  We're going to connect this up to some database or external API that will do things for us, and we are going to reflect the results of this external action in our GUI.  For this simple example, we'll just do this via the message in the Label.  This means that we'll need to implement a framework, in this case I'm going to use my own MVCI (Model-View-Controller-Interactor).  

In the non-FXML version, this is pretty simple.  The first thing I'll do is create a Model:

``` kotlin
class HelloModel {
    val welcomeMessage: StringProperty = SimpleStringProperty("")
}
```

I'll split that `createContent()` method out to be the `build()` method in a `Builder`:

``` kotlin
class HelloViewBuilder(private val model: HelloModel) : Builder<Region> {
    override fun build(): Region = VBox(20.0).apply {
        padding = Insets(20.0)
        alignment = Pos.CENTER
        children += Label().apply {
            textProperty().bind(model.welcomeMessage)
        }
        children += Button("Hello ").apply {
            onAction = EventHandler { model.welcomeMessage.value = "Welcome to JavaFX Application!" }
        }
    }
}
```
Then we'll introduce a Controller:

``` kotlin
class HelloMvciController {
    private val model: HelloModel = HelloModel()
    private val viewBuilder = HelloViewBuilder(model)

    fun getView() = viewBuilder.build()
}
```
And we'll link it back into the Application:

``` kotlin
class HelloNoFXML : Application() {
    override fun start(stage: Stage) {
        with(stage) {
            title = "Hello!"
            scene = Scene(HelloMvciController().getView(), 320.0, 240.0)
            show()
        }
    }
}
```
You can see that this is still fundamentally the same thing.  We've split the contents of the `Label` off into the Model, but this actually improves on the original code because now the `Button` refers to the Model, and has no reference to the `Label`, and we no longer need a variable for it.  This removes internal coupling between the layout `Nodes`.  

But now we need some business logic that will simulate some sort of access to an external API to fetch some data, and this will require an Interactor:

``` kotlin
class HelloInteractor(private val model: HelloModel) {
    fun getWelcome(): Unit {
        model.welcomeMessage.value = "Hello from the Interactor"
    }
}
```
This doesn't actually do anything special, but if you had some code that needed to access an external database or API, this is where you would put it.

To use this, we'll need to update the Controller and the View to handle an "action" triggered by the View (internally from the `Button`).  First the ViewBuilder:

``` kotlin
class HelloViewBuilder(private val model: HelloModel, private val actionHandler : () -> Unit) : Builder<Region> {
    override fun build(): Region = VBox(20.0).apply {
        padding = Insets(20.0)
        alignment = Pos.CENTER
        children += Label().apply {
            textProperty().bind(model.welcomeMessage)
        }
        children += Button("Hello ").apply {
            onAction = EventHandler { actionHandler.invoke() }
        }
    }
}
```
The ViewBuilder now as an additional constructor parameter that takes the Kotlin equivalent of `Runnable`.  It now just invokes the `Runnable` when the `Button` is activated.

And then the Controller:
``` kotlin
class HelloMvciController {
    private val model: HelloModel = HelloModel()
    private val interactor = HelloInteractor(model)
    private val viewBuilder = HelloViewBuilder(model) {interactor.getWelcome()}

    fun getView() = viewBuilder.build()
}
```
Now, when the `Button` is activated, the Interactor's `getWelcome()` method is invoked by the Controller and the Model is updated.  It's important to understand that the `EventHandler` itself is very much a GUI component - it responds to JavaFX events - while the "action" is actually GUI agnostic.  The JavaFX nature of the `EventHandler` has been stripped away, and we're just communicating via a generic Kotlin\Java functional interface.

If we needed to disable and the later re-enable the `Button`, we'd define that in the View, and pass the re-enabling code to the Controller as a `Runnable`.  The original `Runnable` would be transformed to a `Consumer<Runnable>`.  All of this still remains just generic Kotlin\Java function elements.

This is just a silly example, but in real life, the Controller would handle any threading issues, and the Interactor would connect to any external databases or API's to do whatever it needed.

But regardless of what that was, the View wouldn't change at all.  Which is the point of using a framework like MVCI - it keeps the business logic far away from the UI logic.  

As for the FXML...

Theoretically, we should be able to treat the FXML file, the FXMLController and the FXMLLoader as a View in MVCI.  Let's see how that would work.

The `getView()` method in the Controller would now call the FXMLLoader to create the view:

``` kotlin
class HelloMvciControllerFxml {
    private val model: HelloModel = HelloModel()
    private val interactor = HelloInteractor(model)

    fun getView() : Region {
        val fxmlLoader = FXMLLoader(HelloApplication::class.java.getResource("hello-view.fxml"))
        return fxmlLoader.load()
    }
}
```
Then we can update the Application to work the same way that the non-FXML version worked:

``` kotlin
class HelloApplication : Application() {
    override fun start(stage: Stage) {
        stage.title = "Hello!"
        stage.scene = Scene(HelloMvciControllerFxml().getView(), 320.0, 240.0)
        stage.show()
    }
}
```
And it runs just like it did before.

But now it gets more complicated.  We need to get the Model into the FXML Controller, and we need to modify the FXML Controller such that it can call our MVCI controller to perform the action.  The first thing to try is to grab the FXML Controller from the FXMLLoader, and then update it with these references:

``` kotlin
class HelloMvciControllerFxml {
    private val model: HelloModel = HelloModel()
    private val interactor = HelloInteractor(model)

    fun getView(): Region {
        val fxmlLoader = FXMLLoader(HelloApplication::class.java.getResource("hello-view.fxml"))
        val view: Region = fxmlLoader.load()
        val fxmlController = fxmlLoader.getController() as HelloController
        fxmlController.setModel(model)
        fxmlController.setAction { interactor.getWelcome() }
        return view
    }
}
```
Let's look at the FXML Controller:

``` kotlin
class HelloController {
    @FXML
    private lateinit var welcomeText: Label
    private var actionHandler: (() -> Unit)? = null

    fun setModel(model: HelloModel) {
        welcomeText.textProperty().bind(model.welcomeMessage)
    }

    fun setAction(newActionHandler: () -> Unit) {
        actionHandler = newActionHandler
    }

    @FXML
    private fun onHelloButtonClick() {
        actionHandler?.invoke()
    }
}
```
The problem with this approach is that you have to deal with the fact that the Model is not present in the FXML Controller at the time that the FXML Controller is instantiated by the `FXMLLoader`.  This means that you cannot perform setup operations in `initialize()` as you ordinarily would.  In this example, I've loaded all that stuff (well, it's just one thing but you get the idea) into `setModel()`, but that might not be the best approach.

For the action handler, I've taken a different approach and created it as a field in the FXML Controller which is initially `null`.  Then it's set later by the MVCI Controller after the `FXMLLoader.load()` has been invoked.  The downside to this is that every reference to `actionHandler` has to deal with the possible `null` value.  On the upside, the FXML file still implements the `EventHandler` in the standard FXML way and doesn't need to be modified to accommodate the changes to the FXML Controller.

There is a better way.  First, modify the FXML Controller so that it has constructor parameters :

``` kotlin
class HelloController(private val model: HelloModel, private val actionHandler: () -> Unit) : Initializable {
    @FXML
    private lateinit var welcomeText: Label

    @FXML
    private fun onHelloButtonClick() {
        actionHandler.invoke()
    }

    override fun initialize(location: URL?, resources: ResourceBundle?) {
        welcomeText.textProperty().bind(model.welcomeMessage)
    }
}
```
You can see that now we can set the binding in `initialize()` and we don't have to worry about `null` in the `actionHandler` for the `Button`.

Then we have to tell the `FXMLLoader` to use a "Controller Factory" to instantiate the FXML Controller.  This is just a `Callback` that will return an instantiation of our FXML Controller:

``` kotlin
class HelloMvciControllerFxml {
    private val model: HelloModel = HelloModel()
    private val interactor = HelloInteractor(model)

    fun getView(): Region {
        val fxmlLoader = FXMLLoader(HelloApplication::class.java.getResource("hello-view.fxml"))
        fxmlLoader.setControllerFactory { HelloController(model) { interactor.getWelcome() } }
        return fxmlLoader.load()
    }
}
```
This works, and functions exactly the same as the non-FXML version.  

When you look at the code, it's really not *that* complicated.  Just create an FXML Controller with constructor parameters (just like the ViewBuilder in the non-FXML version) and use a `ControllerFactory` to instantiate the FXML Controller inside the `FXMLLoader`.  From that point on, you can treat the FXML Controller as the interface for the View, and presence of FXML should be invisible to the rest of the framework.

You should note that there were no changes at all to the FXML file throughout this example.  This means that, generally speaking, you can design your FXML files with SceneBuilder without any regard to the presence (or not) of an MVC, MVVM or MVCI framework sitting behind it.

You should also note that you're probably not going to see this approach outlined anywhere else.  I spent a bit of time specifically searching for variants on  "JavaFX FXML MVC" and didn't see anything like this anywhere.

Why not?

I think mostly because any tutorial that you see gets sidetracked with the idea that the FXML Controller is the Controller in the MVC sense.  The other thing you always see with this is that the "Model" that they propose is just a POJO of `Observable` classes, and doesn't have any domain/application logic in it.  All the application logic goes into the FXML Controller, which becomes essentially a "God" class.  

# Conclusion

As an experienced JavaFX developer I can say that if I'm building a typical business type application, then the View components (which include both the layout and the GUI logic) comprise a fairly small portion of the code base.  I'd guess something approaching 10% of the total code in the application.

This probably sounds fantastical to most beginners.  Impossible, really.  That's because JavaFX is a huge library which is horribly badly documented for beginners.  Virtually everything that beginners think they are doing right is probably wrong.  They do things like using `EventHandlers` and `ChangeListeners` when `Binding` is the better approach.  Or, things like using `ComboBox.getSelectionModel().getSelectedItem()` instead of just `ComboBox.getValue()`.  

None of these "wrong" approaches is simpler or takes less code than doing it the right way.  In fact, a typical beginner probably writes hundreds of lines of code for every screen that are just not necessary.

So when we talk about "To FXML or not to FXML", we're not talking about some profound decision that dictates the entire structure of your application.  We're talking about how you architect that 10% which comprises the View.  

That is, however, as long as you don't buy into the idea that the FXML Controller acts as the MVC Controller.  Because it's not, and if you treat it like it is then your application structure is going to suffer.  

At the end of the day, the answer to, "Should you use FXML?", depends on whether or not you see enough value in SceneBuilder to overcome the added complexity of dealing with the FXML.  For most beginners, this is probably not true, but they are also probably not going to realize this.  If you are a beginner, and you got this far down this article and you are curious about how you go about creating a layout without FXML, you should read my [Absolute Beginners Guide to JavaFX]({{page.beginners}}).
