---
title:  "Kotlin For JavaFX"
date:   2023-01-28 12:00:00 -0500
categories: kotlin
logo: /assets/logos/JavaFXLogo.png
permalink: /kotlin/kotlin_for_javafx
excerpt: Interested in programming JavaFX in Kotlin?  Here's what you need to know to write JavaFX code that is so much better than anything you can write in Java.
---

# Introduction

I've been using Kotlin for a while now, and I've been using it to write JavaFX applications.  What I've found is that the things that make Kotlin better than Java, make Kotlin way, way better for JavaFX.  

If you're new to Kotlin, you might want to read my [Kotlin for Java Programmers](https://www.pragmaticcoding.ca/kotlin/kotlin_for_java_programmers) article.


Let's get one thing out of the way first...

# Do You Need TornadoFX?

No.

A lot of people are under the impression that TornadoFX is *the* implementation of JavaFX in Kotlin, and that you need it in order to use JavaFX with Kotlin.

This is simply not true.  TornadoFX is an interesting library that does some cool things, but it's **not** any kind of replacement for the JavaFX library.  

So, what is TornadoFX?

More than anything else, it's a wrapper for the standard JavaFX library that attempts to make using JavaFX less verbose and more "Kotlin-like".  It tackles the verbosity by supplying generic "builders" that make use of extension functions on `Node` classes to enable a more declarative style of configuration for layouts.  It also incorporates tools to use an MVC framework, also in a manner to make it as less verbose as possible.

And it's very good at doing this.  But this approach has a couple of issues:

## Repeated Configurations

If you spend any amount of time programming with JavaFX, you're going to find that there are certain patterns of configuration and styling that *you* use over and over and over.  And, if you're using style sheets to handle your styling (as you should), then you're going to be using most of those styles in the same way, over and over and over.  

Want you really want to avoid is repeating that code over and over and over.  Apply DRY (Don't Repeat Yourself), and put that code in a library that you can call.  Any such library that you create is going to be highly personalized to yourself or your team.  

TornadoFX, is all about making it easier to write that code inside your layout.  In many cases your layout code becomes composed of configuring builders that will generate your layout.  The builders are nicely designed to streamline the code, but don't move you towards DRY.  You can probably do it, but it moves your task from writing builders for `Nodes` to writing builders for TornadoFX builders.  

## Configuration and Styling in the Layout

Let's take a look at an example from the TornadoFX documentation, this shows a call to one of their builders:

``` kotlin
tableview<Person> {
    items = persons
    column("ID", Person::idProperty)
    column("Name", Person::nameProperty)
    column("Birthday", Person::birthdayProperty)
    readonlyColumn("Age", Person::age).cellFormat {
        text = it.toString()
        style {
            if (it < 18) {
                backgroundColor += c("#8b0000")
                textFill = Color.WHITE
            }
        }
    }
}
```
This would become your layout code with TornadoFX.  You can see how this strips away a lot of the boilerplate that you'd need with a pure JavaFX implementation, and the result absolutely does do more with less.  

But that "Age" column.

Yes, the customization of the `Cell` is way more straightforward than in pure JavaFX, but it's still a clear violation of the "Single Responsibility Principle" - it should be delegated.   

But the design of TornadoFX encourages you to put the configuration and styling in with the layout.

So, the question is whether or not the streamlining of the code compensates for the complication of having the configuration and styling mingled in with the layout.

## Should You Use TornadoFX?

If you're following the ideas in this website - building Reactive layouts with MVCI and sticking to the "Single Responsibility Principle" and DRY - there's probably not a lot of benefit to using TornadoFX.

TornadoFX tries to deal with the complexity by simplifying the integration of the configuration into the layout code.  What it doesn't do is *remove* the configuration from your layouts, which, in my opinion, is a better approach.  

# Kotlin JavaFX Techniques

There are a number of cool features in Kotlin that make JavaFX a lot easier to implement, and some features that mean you need to take a look at how you should build things in Kotlin.  Let take a look at some of these...

## The .apply{} Function

This is the star in Kotlin for JavaFX.  When you call `Object.apply{}` it passes the `object` to the lambda in the `{}` as `this`, and `apply{}` returns the original `object`.  This is especially useful in the pattern `val x = Node().apply{}`.

Let's say you've got a `Label` that you want have display some data the screen and bound to a `Property` in your model.  You have a style class set up for, too.

In Java:

``` java
Label label = new Label();
label.getStyleClass().add("data-label");
label.textProperty().bind(model.someDataProperty());
hbox.getChildren().add(label);
```

But in Kotlin, you can use `.apply{}` and avoid instantiating `label` altogether:

``` kotlin
hbox.children += Label().apply{
  styleClass += "data-label"
  textProperty().bind(model.someData)
}
```
(Note that, just like with a class in Java, you can skip the `this.` when referring to member fields and functions)

Now, that might not seem like much, but it does lead to things like this:

``` kotlin
hBox = HBox(10.0,
  Label("Name: ").apply{styleClass += "prompt-label"},
  Label().apply{
    styleClass += "data-label"
    textProperty().bind(model.someData)
  })
```

This can get a bit garbled (something to watch out for with `.apply{}`), so you could try:

``` kotlin
hBox = HBox(10.0).apply{
  children += Label("Name: ").apply{
    styleClass += "prompt-label"
  }
  children += Label().apply{
    styleClass += "data-label"
    textProperty().bind(model.someData)
  }
}
```    
Perhaps even more useful is that you create functions using the same technique:

``` kotlin
hBox = HBox(10.0).apply{
  children += listOf(Label("Name: ").apply{styleClass += "prompt-label"}, dataLabel(model.someData))
  styleClass += "red-border-box"
}

fun dataLabel(boundValue : ObservableStringValue) = Label().apply{
  styleClass += "data-label"
  textProperty().bind(boundValue)
}

```

## JavaFX Property Fields

Kotlin uses the term "property" to refer to what would be called "fields" in Java.  Kotlin properties are actually more complicated structures that have integrated getters and setters (which can be customized) which are automatically called when you access the property.  This means that you can directly reference the property from outside the class without worrying about tying down your internal implementation.

Of course, in JavaFX a `Property` is specific kind of `ObservableValue`, which is quite different from a Kotlin property.  Additionally, there is the concept of a `Property Bean`, which is a particular structure exposing the `Property` outside its containing class.  

A `Property Bean` has three methods in its containing class.  For a `Property` called `fred` you would have the following methods:

1. A getter for the `Property` value called `getFred()`.
1. A setter for the `Property` value called `setFred()`.
1. A getter for the property itself called `fredProperty()`

In Kotlin, you can't just write the `getFred()` and `setFred()` methods because they would conflict with the internal getter and setter for `fred`.  So you have to do something a little bit different.  I wrote an entire [article](https://www.pragmaticcoding.ca/javafx/kotlin_properties) about this a while back.  

Here's a sample program from the original article, showing how this is done.

{% gist f6d3bd9dfc261b6d411d7945172d5833 %}

And the output looks like this:

```
Nickname: Shorty
Property: ObjectProperty [value: Shorty]
```

That gets the job done, and it allows you direct access to the values of the `Property` via `Class.fred`.  

The JavaFX `Property Bean` is the accepted way to implement `Properties` in a Model, but...

### Do you really need Property Beans?

This naming convention is primarily designed to facilitate those tools that use reflection in the class `PropertyReference` to access fields via a `String` name.  In the standard JavaFX library, this class is used in the following places: `PropertyValueFactory`, `TreeViewPropertyValueFactory` and variants of `Bindings.select()`.  The first two should probably be deprecated, because you shouldn't use them any more now that we have lambdas and writing the equivalent `Callback` is trivial.

The last case is about 14 methods in the `Bindings` class that, "Create a binding used to get a member, such as a.b.c.", according to the JavaDocs.  Personally, I have never used `Bindings.select()`, and I suspect that it's primarily intended when you want to pass around `Property` names as `Strings` and then directly use those names for binding.  

Of course, if you're using external JavaFX libraries, they may require that you use the `Property Bean` format because they are using something that incorporates `PropertyReference`.  

But...if you're not doing any of those things, you can ditch the `Property Bean`.  Just make your `Property` property a `val` you're done.

Personally, I've stopped habitually using the `Property Bean` pattern because it just doesn't add any value for me.

## Extension Functions, Package Functions and Operator Overloading

The next few features go together produce some really powerful techniques which are the core tools you can use to compress your layout code.

### Extension Functions

One of the coolest features of Kotlin is the ability to extend a class without creating a new class.  With this technique you can add both methods and data to an existing class without having access to the source of that class.

For instance, you can do something like this:

``` kotlin
fun Int.plusThree() : Int = this + 3
   .
   .
   .
   val x: Int = 7
println("The answer: ${x.plusThree()}")   
```

This can be amazingly valuable with JavaFX, where there are a lot of classes that you use all the time where it would be handy to have methods that do something with a single call that normally take a few steps.

Extension functions can also be used as decorators using `.apply{}`:

``` kotlin
private fun createContent(): Region = BorderPane().apply {
   val model: StringProperty = SimpleStringProperty("Albert")
   padding = Insets(20.0)
   center = HBox(10.0, Label().bound(model), Label("George"))
               .styled("test-border3")
               .padded(10.0)
               .aligned(Pos.CENTER)
}
}

fun <T : Node> T.styled(newStyleClass: String): T = apply { styleClass += newStyleClass }

fun <T : Region> T.padded(padSize: Double): T = apply { padding = Insets(padSize) }

fun <T : Labeled> T.bound(otherProperty: ObservableValue<String>): T = apply { textProperty().bind(otherProperty) }

fun HBox.aligned(pos: Pos): HBox = apply { alignment = pos }
```

For this, however, it's important to remember that extension functions are evaluated statically.  This means that if you want a decorator to return the original type for any of the subclasses of a type you need to use the `<T : Class> T.` declaration.  In this example, the `setAlignment()` method is included at the `HBox` class itself, so you can declare the decorator directly at `HBox`; `getStyleClass()` is defined at the `Node` level, so `<T: Node> T` is used; and `setPadding()` is defined at the `Region` level, so `<T: Region> T` is used.  


### Package Functions

Another cool feature in Kotlin is "Package Functions".  These are methods that aren't part of any specific class, they're just functions attached to a package.  In this way they are very similar to `static` methods in Java, but you don't have the baggage of having to call them with `ClassName.staticMethod()` format.  This makes a dramatic difference in how the resulting code looks:

``` kotlin
fun labelOf(boundValue : StringProperty) : Label = Label().apply {textProperty().bind(boundValue)}
   .
   .
   .
val hbox = HBox(10.0, Label("Name:"), labelOf(model.nameProperty))
```

### Operator Overloading

[Operator Overloading](https://kotlinlang.org/docs/operator-overloading.html) is another nifty idea that Kotlin brings to the table which can simplify your layout code.  The idea is that a specific set of operators are automatically associated with specific functions if they are defined in a class as `operator`.  So the "+" operator is associated with `class.plus()` if it exists.  

But here's the cool part:  The operator functions can be extension functions!

This means that you can add operator overloads to any of the JavaFX classes.  Let's see how this works.  To me, the "+=" operator seems like a good fit to represent binding on a Property.  Here's how you would do that:

``` kotlin
operator fun StringProperty.plusAssign(otherProperty: StringProperty) = this.bind(otherProperty)
   .
   .
   .
val model : StringProperty = SimpleStringProperty("ABC")
val label = Label()
label.textProperty() += model
```
Or, if you prefer...
``` kotlin
operator fun Labeled.plusAssign(otherProperty: StringProperty) = this.textProperty().bind(otherProperty)
   .
   .
   .
val model : StringProperty = SimpleStringProperty("ABC")
val label = Label()
label += model
```

### Infix Functions

If a function of a class has a single parameter, then you can declare it as [Infix](https://kotlinlang.org/docs/functions.html#infix-notation).  This means that you can drop the "." and the brackets when making the call.  The result is something that looks very declarative.  This is especially true if you implement decorator functions as `infix`:

``` kotlin

infix fun <T : Labeled> T.styledAs(labelStyle: String) = apply { styleClass += labelStyle }

infix fun <T : Labeled> T.boundTo(value: ObservableStringValue) = apply { textProperty().bind(value) }

fun testIt(): Unit {
   val fred: StringProperty = SimpleStringProperty("abc")
   val label1 = Label() styledAs "label-prompt" boundTo fred
   val label3 = Label().styledAs(LabelStyle.PROMPT).boundTo(fred)
}
```

The first two methods are declared as `infix`, but you don't have to use them that way.  These are all also decorator style methods that return the enclosing object.  That means that you can string them together. The first calling example shows how this works, and the second shows it without the infix calling style. The two versions are functionally equivalent.  

You have to be a little bit careful with the order of operations with infix, but chaining the decorators together like this works nicely.

### Putting it All Together

If you're looking to create a standard toolkit that you can use over a variety of projects, then your best approach is to create Kotlin files in their own package to house your functions.  You can call it `FxExtentions.kt` or `ExtensionsFX.kt` or `WidgetsFX.kt`.  You don't need to have a class of the same name, you can just drop all of your package functions and extension functions in there as you like.  You can even add some custom classes, Kotlin's cool with that.  

This will get all this stuff right out of your layout code, and out of your mind - where you don't want it.  

My approach is to create a complementary CSS file to go with the code and then write the various utility functions to work with that CSS file.  

Naming becomes very important, and you have to think about how the code will read using the names that picked.  This could also depend on how you intend to use the utilities.  Consider this:

``` kotlin
val label = Label() styledAs LabelStyle.DATA boundTo model.someDataProperty
```
Here the decorators have all been name as past-tense verbs, essentially declaring their effects.  I think this reads nicely as a description of the `Node`; it's a `Label` styled as data and bound to a property.  

However, you could use this:

``` kotlin
val label = Label() styleAs LabelStyle.DATA bindTo model.someProperty
```
Now the decorators read as commands.  This version is more declarative, and not descriptive.

But, if you don't like the `infix` usage, then you would have:

``` kotlin
val label = Label.styledAs(LabelStyle.DATA).boundTo(model.someDataProperty)
```
or...
``` kotlin
val label = Label.styleAs(LabelStyle.DATA).bindTo(model.someDataProperty)
```
Since we're used to traditional function call being named as actions, the second version feels more natural.  But both ways could possibly work.

#### Utility Library Files

With these techniques you can create a toolkit that will make your Kotlin layout code look much more streamlined and declarative than Java code.  The trick is to find all the "pain points" in your layout code, the places that you get bogged down in details, and move them into your library of functions.  

Here's what I came up with in just a few minutes.  First, `WidgetFX.kt`:
``` kotlin
package ca.pragmaticcoding.widgetsfx

fun Scene.addWidgetStyles() = apply {
   object {}::class.java.getResource("widgetsfx.css")?.toString()?.let { stylesheets += it }
}

fun <T : Parent> T.addWidgetStyles() = apply {
   object {}::class.java.getResource("widgetsfx.css")?.toString()?.let { stylesheets += it }
}

enum class TestStyle(val selector: String) {
   BLUE("test-blue"), RED("test-red"), GREEN("test-green")
}

infix fun <T : Node> T.testStyleAs(nodeStyle: TestStyle) = apply { styleClass += nodeStyle.selector }

infix fun <T : Region> T.padWith(padSize: Double): T = apply { padding = Insets(padSize) }

infix fun <T : Node> T.addStyle(newStyleClass: String): T = apply { styleClass += newStyleClass }

fun textFieldOf(value: StringProperty) = TextField().apply { textProperty().bind(value) }

infix fun TextField.bindTo(value: StringProperty) = apply { textProperty().bind(value) }

fun buttonOf(text: String, handler: EventHandler<ActionEvent>) = Button(text) addAction handler

operator fun Pane.plusAssign(newChild: Node) {
   children += newChild
}

infix fun HBox.alignTo(pos: Pos): HBox = apply { alignment = pos }

infix fun <T : ButtonBase> T.addAction(eventHandler: EventHandler<ActionEvent>): T = apply { onAction = eventHandler }
```
This is just a grab-bag of functions and utilities that aren't necessarily related to each other.

There are decorators to add the `widgetsfx.css` stylesheet to either a `Scene` or a `Parent`.  Since these are declared at a package level, there's no class to provide `getResource()` to find the stylesheet.  Instead the function declares an anonymous `Object` that will have the correct package which can provide the `getResource()` method.

`Region.padWith()` is super useful.  It gets absolutely tedious writing `setPadding(new Insets(10))` over and over and over.  The vast majority of the time, you just want all of the sides to have the same padding so a function that just takes a single `Double` parameter makes a huge difference.

and then `Labels.kt`
``` kotlin
package ca.pragmaticcoding.widgetsfx

enum class LabelStyle(val selector: String) {
   PROMPT("label-prompt"), HEADING("label-heading")
}

infix fun <T : Labeled> T.styleAs(labelStyle: LabelStyle) = apply { styleClass += labelStyle.selector }

infix fun <T : Labeled> T.bindTo(value: ObservableStringValue) = apply { textProperty().bind(value) }

fun promptOf(value: ObservableStringValue) = Label() styleAs LabelStyle.PROMPT bindTo value
fun promptOf(value: String) = Label(value) styleAs LabelStyle.PROMPT

fun headingOf(value: String) = Label() styleAs LabelStyle.HEADING

operator fun Labeled.plusAssign(otherProperty: StringProperty) = run { textProperty() += otherProperty }
```
For this file, I've grouped together all of the functions that deal with `Labels` strictly as an organizational tool.

In both of these files, I've declared an `Enum` to abstract the CSS selector classes so that they don't need to be known outside the utility files.   

Obviously, these functions require a complementary style sheet to work with:

``` css
.root {
   -theme-colour: #113969;
   -contrast-colour: #C6522F;
   -complementary1: #43AA8B;
   -complementary2: #B4CDED;
   -complementary3: #80A4ED
}


.test-blue {
  -fx-border-color: blue;
  -fx-background-color : lightcyan;
}

.test-red {
  -fx-border-color: firebrick;
  -fx-background-color : lavenderblush;
}

.test-green {
  -fx-border-color: green;
  -fx-background-color : derive(yellowgreen, +50%);
}

.label-prompt {
  -fx-text-fill: -theme-colour;
  -fx-font-weight: bold;
  -fx-font-size: 15px;
}

.label-heading {
  -fx-text-fill: -contrast-colour;
  -fx-font-weight: bold;
  -fx-font-size: 32px;
}
```
#### Using It

Good layout code should make it easy for any reader to understand at a glance how a layout works.  This means keeping the configuration out of the way, as this is what causes confusion.  But how you use these library functions can make a big difference to readability.

For instance, this is hard to read (horrible, actually):

``` kotlin
private fun createContent(): Region = BorderPane().apply {
   top = headingOf("Test Screen")
   center = VBox(20.0).apply {
      children += HBox(10.0).apply {
         children += Label("Name") styleAs LabelStyle.PROMPT
         children += TextField() bindTo nameProperty
      } padWith 10.0 alignTo Pos.CENTER_LEFT
      children += Button("Click Me") addAction { buttonAction() }
   }
} testStyleAs TestStyle.BLUE padWith 20.0
```

and this is maybe a bit easier, but still too difficult:
``` kotlin
private fun createContent(): Region = (BorderPane() testStyleAs TestStyle.BLUE padWith 20.0).apply {
    top = headingOf("Test Screen")
    center = VBox(20.0,
                  HBox(10.0,
                       Label("Name") styleAs LabelStyle.PROMPT,
                       TextField() bindTo nameProperty) padWith 10.0 alignTo Pos.CENTER_LEFT,
                  Button("Click Me") addAction { buttonAction() })
 }
```
but specific builders work best, so this is probably clear enough:

``` kotlin
private fun createContent(): Region = BorderPane().apply {
   top = headingOf("Test Screen")
   center = VBox(20.0, createNameRow(), buttonOf("Click Me") { buttonAction() })
   this testStyleAs TestStyle.BLUE padWith 20.0
}

private fun createNameRow() =
   HBox(10.0, promptOf("Name"), textFieldOf(nameProperty)) padWith 10.0 alignTo Pos.CENTER_LEFT
```
But this is probably best of all:

``` kotlin
private fun createContent(): Region = BorderPane().apply {
   top = headingOf("Test Screen")
   center = VBox(20.0, createNameRow(), createButton())
} testStyleAs TestStyle.BLUE padWith 20.0

private fun createButton() = buttonOf("Click Me") { buttonAction() }

private fun createNameRow() =
   HBox(10.0, promptOf("Name"), textFieldOf(nameProperty)) padWith 10.0 alignTo Pos.CENTER_LEFT
```
There's no doubt that this last version achieves the "at a glance" objective of the layout code.  You can see that the `BorderPane` has the top and centre occupied, and you can see that the centre has two rows in it.  If you're curious about the "Name" row, or the `Button` you can click through to them in the code.  This is really no different than the best approach in Java, but the `apply{}`, the builders, and the `infix` functions strip out all of the extraneous boilerplate that would otherwise bloat the layout code.  

## Coroutines

Coroutines are a neat feature that allow you to run multiple blocking operations on a single thread.  When one task blocks, it doesn't halt the thread, but is "parked" and the thread is freed up to run other tasks.  As each task blocks, Kotlin can then "un-park" a previously parked task that has become unblocked, and run it on the thread until it either completes or becomes blocked again.  

There is a special library (part of the Kotlin companion libraries for coroutines) that allows you to run coroutines on the FXAT.  So that you can do something like this:

``` kotlin
private fun createContent(): Region = BorderPane().apply {
    padding = Insets(20.0)
    center = VBox(20.0,
                  HBox(10.0, Label().bound(model))
                     .styled("test-border3")
                     .padded(10.0)
                     .aligned(Pos.CENTER),
                  Button("ClickMe").apply { onAction = EventHandler { _ -> buttonAction() } })
 }

 private fun buttonAction() {
    GlobalScope.launch(Dispatchers.JavaFx) {
       println("Starting the function")
       counter += 1
       model.value = counter.toString()
       delay(10000)
       println("Ending the function")
       counter += 100
       model.value = counter.toString()
    }
 }
```
The `Dispatchers.JavaFX` tells it to launch the coroutine on the FXAT.  Because of this, you can update `model` even though it's bound to a screen element.  The call to `delay()` is used to simulate a blocking operation.  While the `delay()` is running, you can continue to use the GUI, and you can, in fact, click the `Button` again and launch another coroutine on the FXAT.  

As you can see, running the code as a coroutine simply requires wrapping it in a call to `GlobalScope.launch` with the correct dispatcher.  It's really that trivial.  

Potentially, you could do any number of blocking operations that you'd normally use `Task` for using coroutines and it should work just fine.  However you should keep a couple of things in mind:

1.  This is only for blocking operations, like accessing an external API over a network.  Running a very long process that simply takes a long time to complete is still going to hang your GUI.

1.  Using this technique introduces the possibility of some of the concurrency issues that the single threaded FXAT is designed to avoid.  This particular example is designed to demonstrate that each click of the button messes with the value of `counter`.  If you had code that assumed that counter was equal to `101` after completion, it could fail.  

To expand on the "concurrency issues" part:  If you ran the same code without using coroutines, then the GUI would definitely hang for 10 seconds after you click the button.  No jobs would get processed of the FXAT and no events would get handled.  But you could guarantee that the state of the GUI would be exactly the same after the delay as it was before the `delay()`, and this includes `model.value` and `counter`.  But with coroutines, when the execution continues after the `delay()` there's no reason to believe that these things won't have changed.  

Coroutines are generally considered an advanced technique in Kotlin, and are really included here for the sake of completeness.  There are places where they can be of value, but you should probably continue to use `Task` for most situations.
