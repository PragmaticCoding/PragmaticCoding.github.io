---
title:  "More Beginner Mistakes"
date:   2024-04-30 00:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/beginner_mistakes2
Diagram: /assets/posts/MVCI.png
Dispatch1: /assets/elements/EventDispatchChain1.png
Dispatch2: /assets/elements/EventDispatchChain2.png
DemoRunning: /assets/posts/MvciDemoRunning.png
DemoDone: /assets/posts/MvciDemoDone.png
Chain1: /assets/elements/Chain1.png
Chain2: /assets/elements/Chain2.png
excerpt: Another look at some beginner code, and how it could be improved.
---

# Introduction

After literally a decade writing JavaFX applications, it can be hard to remember how I thought things were supposed to be done when I started out.  Thankfully, there seems to be a nearly endless supply of beginners having trouble, posting their code and giving us a glimpse into their thought processes.  

In this article we are going to look at some beginner code that really highlights how much more complicated JavaFX is when you don't know a lot about it.

Even after getting this code to compile and run, it's still not clear to me exactly what it's supposed to do.  In general terms, it displays a screen with a title and three `Buttons`, one of which is supposed to hide some of the `Buttons` while displaying a `Label\TextField\Button` combination to allow some user entry.  This data entry display also contains a "Back" `Button` to go back to the original, three `Button` display.

While the general idea of the application is easy to figure out, details of how the screens are supposed to look are a bit murky because the approach taken, and the fact that it wouldn't even compile in its original state.  I've taken a "best try" approach when refactoring it to come up with something that seems to hold to the original spirit, if not the fine details of the intended layout.

My version is in Kotlin.  It's pretty small, so you shouldn't have much trouble following it and the main point of this article is to highlight general concepts and JavaFX techniques - none of which are language dependent.  

# The Code

``` java
public class opening extends Application {
	// first scene (buttons
	Label gameTitle = new Label("Basil's Cat Game :3");

	// new game buttons
	TextField putNameHere = new TextField();

	Label putNameTxt = new Label("Name:");
	Button nameOkButton = new Button("OK");

	VBox titleVBox = new VBox();
	HBox titleHBox = new HBox();
	GridPane titleGroup = new GridPane();
	Button backBtn = new Button("BACK");

	VBox title = new VBox();
	Button exitBtn = new Button();

	Group titleGroupSec = new Group();

	boolean buttonClicked = false;

	@Override
	public void start(Stage primaryStage) throws Exception {
		// needs buttons to select what to do and also a text field to store a name
		gameTitle.setFont(Font.font(STYLESHEET_CASPIAN, 40));

		Button playBtn = new Button("NEW");
		playBtn.setFont(Font.font(30));
		playBtn.setBackground(Background.EMPTY);
		Button loadBtn = new Button("LOAD");
		loadBtn.setFont(Font.font(30));
		loadBtn.setBackground(Background.EMPTY);
		Button exitBtn = new Button("EXIT");
		exitBtn.setFont(Font.font(30));
		exitBtn.setBackground(Background.EMPTY);

		Button exitBtnSec = new Button("EXIT");
		exitBtnSec.setFont(Font.font(30));
		exitBtnSec.setBackground(Background.EMPTY);

		VBox btnVBox = new VBox();
		btnVBox.getChildren().add(0, gameTitle);
		btnVBox.getChildren().add(1, playBtn);
		btnVBox.getChildren().add(2, loadBtn);
		btnVBox.getChildren().add(3, exitBtn);
		btnVBox.setAlignment(Pos.TOP_CENTER);
		btnVBox.setSpacing(20);

		titleGroup.getChildren().remove(titleHBox);
		titleGroup.getChildren().remove(titleVBox);
		titleGroup.getChildren().addLast(btnVBox);
		titleGroup.setAlignment(Pos.TOP_CENTER);
		titleGroup.setPadding(new Insets(50));

		/**
		 * maybe should change to say "NEW"? anyways, make them pick a name (move name
		 * feature to here & show a quick text based tutorial that does some waiting
		 * liek im sayin it to the player :3
		 *
		 * make them roll for a cat nd name the cat. save the cat and it's features to a
		 * file that is specific to this player.
		 *
		 * make it so they can access a list to see player info (name) and cat info(how
		 * many cats, names, breeds, traits, colors)
		 */
		playBtn.setOnMouseClicked(playBtnClickedEvent -> {
			btnVBox.getChildren().remove(loadBtn);
			btnVBox.getChildren().remove(exitBtn);
			btnVBox.getChildren().remove(playBtn);



			exitBtn.setAlignment(Pos.TOP_LEFT);

			gameTitle.setFont(Font.font(STYLESHEET_CASPIAN, 40));

			putNameTxt.setFont(Font.font(STYLESHEET_CASPIAN, 20));

			putNameHere.setEditable(true);

			nameOkButton.setFont(Font.font(10));

			title.getChildren().add(putNameTxt);
			title.getChildren().addLast(exitBtn);
			title.getChildren().addFirst(gameTitle);

			title.setSpacing(10);
			title.setAlignment(Pos.TOP_CENTER);

			titleHBox.getChildren().addFirst(putNameHere);
			titleHBox.getChildren().addLast(nameOkButton);
			titleHBox.setPadding(new Insets(100));
			titleHBox.setAlignment(Pos.CENTER);

			titleGroup.getChildren().addFirst(backBtn);
			titleGroup.getChildren().add(title);
			titleGroup.getChildren().addLast(titleHBox);



		});

		/**
		 * here, u will probably have to implement something more complex than a file
		 * writer if you decide to use photos or graphics to make ur kitties :P
		 *
		 * call the names of players (only players that have pressed the PLAY button &
		 * gotten a cat.. show the amt of cats they have next to their name :P)
		 *
		 * if they click on the button that has their name/amt of cats then make open
		 * that save file in the stage
		 */
		loadBtn.setOnMouseClicked(loadBtnClickedEvent -> {

		});

		exitBtn.setOnMouseClicked(exitBtnClickedEvent -> {
			primaryStage.close();
		});

		Scene scene = new Scene(titleGroup, 500, 500);
		primaryStage.setScene(scene);
		primaryStage.setTitle("love u <3");
		primaryStage.show();

	}

	public static void main(String[] args) {
		launch(args);
	}
}
```

# The Fundamentals

## Breaking the Single Responsibility Principle (SRP)

SRP and DRY (see below) are the two foundational principles for writing good code.  Yet beginners violate this all the time.

Every block of code should be directly responsible for one thing, and one thing only.  Every method should be directly responsible for one, and one thing only.  Every class should be directly responsible for one thing, and one thing only.  Do you see a pattern here?

In this program we have one class that does everything, and one 130 line method that does almost everything.  This is a clear indication that SRP has been violated.

## Breaking "Don't Repeat Yourself" (DRY)

Take a look at this snippet from the code:

``` java
Button playBtn = new Button("NEW");
playBtn.setFont(Font.font(30));
playBtn.setBackground(Background.EMPTY);
Button loadBtn = new Button("LOAD");
loadBtn.setFont(Font.font(30));
loadBtn.setBackground(Background.EMPTY);
Button exitBtn = new Button("EXIT");
exitBtn.setFont(Font.font(30));
exitBtn.setBackground(Background.EMPTY);
```
Here we have the same three lines of code repeated three times.  The only differences are in the variable that is created and the text in the `Button`.  

Why is this bad?  First, it clutters things up.  There's now 9 lines of code that you have to read carefully to determine that all three `Buttons` are essentially configured the same.  Secondly, if you want to change something, you now have to do it 3 times.  

Finally, it's error prone.  One of the most common places for making silly mistakes that way more time to track down than you'd expect comes from repeated blocks of code.  Take a look at this:

``` java
Button playBtn = new Button("NEW");
playBtn.setFont(Font.font(30));
playBtn.setBackground(Background.EMPTY);
Button loadBtn = new Button("LOAD");
loadBtn.setFont(Font.font(30));
playBtn.setBackground(Background.EMPTY);
Button exitBtn = new Button("EXIT");
exitBtn.setFont(Font.font(30));
exitBtn.setBackground(Background.EMPTY);
```
It's almost the same block of code, but it has an error in it.  The kind of error that you get from cutting and pasting repeatedly.

You should make it a habit to **never** cut and paste from your own code.  Maybe from StackOverflow, but cutting and pasting from your own code almost guarantees that you're breaking DRY, and you should just stop doing that.

## Violating YAGNI and Clinginess

Programmers always seem to want to over-engineer and future-proof everything.  Yet the truth is that most of the time, "You Ain't Gonna Need It" (YAGNI).  And then, once they've built something they don't need they won't use the `Delete` key on it - clinginess.  

In this example we have both `titleGroupSec` and `buttonClicked` which are instantiated as fields, but never used.  In the end this just clutters things up and causes confusion.


## Excessive Coupling

Coupling is the thing that will really complicate your project.  Both SRP and DRY are huge helpers in reducing coupling, which is another good reason to apply them.  

The main tool in reducing coupling is to give every element in you application the smallest possible scope.  Define variables inside of blocks of code when you can, and use local variables instead of fields whenever possible.  


## Bad Names

Yes, naming things well is hard.  Get over that and just do it.

Avoid the temptation to be cute or clever with your names.  We all go through that phase.  I know that I did, and I'm pretty sure that none of my coworkers appreciated it.  

In this code we have:

``` java
titleHBox.getChildren().addFirst(putNameHere);
```

What is `putNameHere`?  You really cannot tell from this code.  It's a `Node` subclass or it couldn't be put into an `HBox`, but other than that, we don't know.  Yes, you can put the cursor over it in any modern IDE and it will tell you what it is.  But you still can't just read the code an know.  I can appreciate that `putNameHere` is cute, but it's not very clear and, therefore, not a very good name.

I'm not against the idea of putting an indicator of the `Node` type in the name, as in `titleHBox` in the snippet.  Just be sure to change the name if you change the `Node` type.  We also have this in the code:

``` java
GridPane titleGroup = new GridPane();
```
Presumably, `titleGroup` was originally defined as a `Group` and later changed to a `GridPane`.  Looking at the code where it was used, I had no idea it was a `GridPane` and this had me confused for a while.

# JavaFX Principles

Now we move on to JavaFX principles that you should try to follow, and how you can get into trouble if you don't.

## Changing Layouts

I have written thousands and thousands of lines of JavaFX layout code, and I can hardly remember ever writing any code to change the layout of a screen while it was running.  But beginners love to do this.  I guess it just seems like the natural way to do things.  

In this particular example, the desire to change the layout "on the fly" leads to virtually all of the other problems with the code.  

In particular, this is not a thing:

``` java
titleHBox.getChildren().addFirst(putNameHere);
titleHBox.getChildren().addLast(nameOkButton);
```
This just created a bunch of red squiggly lines in my IDE.  I think that these methods, `addFirst()` and `addLast()` are for `LinkedLists`, which the `ObservableLists` from `Pane.children` are not.  

However, the need to put stuff in at the beginning of a list of `children` is really only something you'll need if you're messing around with a running layout.  There's also lots of calls to `List.remove()`, which does work, but shouldn't ever be needed.  

Part of the problem is that in order to do all of this swapping around, you need to have references to the various `Nodes` that you're going to be adding and removing on the fly.  And, since you're going to be doing all of the swapping from different places in the layout, these references need to be global throughout all of the layout code.  This creates a large amount of coupling.  A really large amount.

In this code, we have two places where the swapping would occur.  One is in the action for `playBtn`, and the other would be in the `backBtn` action.  However, there is no code included for the `backBtn` action, probably because putting everything back was a daunting task that the programmer couldn't bring themselves to face yet.  Both these action handlers would have to have access to references to the containing layout class and virtually all of the children.  

## Not Keeping the Layout Understandable

You should be able to look at a piece of layout code and get an idea in your head about how it's going to look when it's running.  If you can't, then something has gone seriously wrong.  

In general, following SRP and DRY is going to force you to break your layout into chunks, and put those chunks together in a manner that makes it clear how the layout should look.

In this example we can see a `Button` instantiated and configured in one section of code, added to a layout 20 lines later, and then getting its action defined 50 lines after that.  This means that you have to read through almost all of the layout code to understand how an element is configured and added to the layout.  

## Not Using the Correct Layout Classes

Perhaps this rule should just be called "Don't Abuse `AnchorPane` and `GridPane`".  `AnchorPane` is the most abused layout class, perhaps because it's alphabetically first in the choices in SceneBuilder and beginners think it's some kind of default.  Then they never use it as an `AnchorPane` and use `setLayoutX()` and `setLayoutY()` (by dragging stuff in SceneBuilder) to position the children.

And for some reason, beginners think that everything else needs to be a `GridPane`.  I don't know why.  

Here we have `titleGroup` which is actually a `GridPane` but it`s not used like a `GridPane`.  Children are dumped in, but never assigned to rows or columns.  I was a little tempted to see what that actually does, but then I decided I didn't need to know.  I do know that when I ran the original code, it looked a little confused.

## Not Using the Correct Event Types

This code uses the `MouseClickedEvent` for all of the `Button` actions.  This *will* work for mouse clicks, but won't work for keyboard access.  So if the user tabs around the screen to put the focus on a `Button` and hits `Enter`, it's not going to work.


# The New Version

The new version is in Kotlin.  Let's take a look at it first:

``` kotlin
class Opening : Application() {
    private val isShowingNewGame: BooleanProperty = SimpleBooleanProperty(false)

    override fun start(primaryStage: Stage) {
        primaryStage.scene = Scene(createContent(), 500.0, 500.0).apply {
            Opening::class.java.getResource("beginners.css")?.toString()?.let { stylesheets += it }
        }
        primaryStage.title = "love u <3"
        primaryStage.show()
    }

    private fun createContent(): Region = BorderPane().apply {
        top = HBox(Label("Basil's Cat Game :3") styleWith "title-label").centred()
        bottom = HBox(createButton("EXIT") { Platform.exit() }).centred()
        center = StackPane(buttonBox() showWhen isShowingNewGame.not(), newGameBox() showWhen isShowingNewGame)
    }

	  private fun buttonBox() =
        VBox(20.0, createButton("NEW") { isShowingNewGame.value = true }, createButton("LOAD") {}).centred()

    private fun newGameBox(): Region = BorderPane().apply {
        center = VBox(
            HBox(6.0, Label("Name:") styleWith "prompt-label", TextField(), Button("OK")).centred()
        ).centred()
        top = Button("Back").apply { setOnAction { isShowingNewGame.value = false } } styleWith "transparent-button"
    }

    private fun createButton(title: String, action: EventHandler<ActionEvent>) = Button(title).apply {
        onAction = action
        styleWith("transparent-button")
    }
}

infix fun <T : Node> T.styleWith(selector: String): T = apply { styleClass += selector }
infix fun <T : Node> T.showWhen(observed: ObservableBooleanValue): T = apply {
    visibleProperty().bind(observed)
    managedProperty().bind(observed)
}

fun HBox.centred() = apply { alignment = Pos.CENTER }
fun VBox.centred() = apply { alignment = Pos.CENTER }


fun main() {
    Application.launch(Opening::class.java)

```

You'll see that pretty much all of it is still inside `Opening` which extends `Application`.  It was left this way because the application really doesn't "do" anything, so there was no need to create a framework for the application.  It would be easy, though, to replace `createContent()` with `OpeningController.getView()` and introduce an MVCI framework.

## Application.start()

`Application.start()` should only deal directly with the mechanics of `Stage` and `Scene`.  If you put anything else in here, you're doing it wrong.  The original code had **everything** in `Application.start()`, which was an extreme violation of SRP:

``` kotlin
override fun start(primaryStage: Stage) {
		primaryStage.scene = Scene(createContent(), 500.0, 500.0).apply {
				Opening::class.java.getResource("beginners.css")?.toString()?.let { stylesheets += it }
		}
		primaryStage.title = "love u <3"
		primaryStage.show()
}
```
This is short and sweet.  We create the `Scene` and delegate the creation of the layout to `createContent()`.  Then we attach the StyleSheet, and put it into the `Stage`.  Then we configure the title and put the `Stage` on the screen.  

That's it.  That's all this method should ever do.

## The createContent() Method

The entire layout comprises about 20 lines of code across 4 methods.  Let's look at the top level method, `createContent()`:

``` kotlin
private fun createContent(): Region = BorderPane().apply {
		top = HBox(Label("Basil's Cat Game :3") styleWith "title-label").centred()
		bottom = HBox(createButton("EXIT") { Platform.exit() }).centred()
		center = StackPane(buttonBox() showWhen isShowingNewGame.not(), newGameBox() showWhen isShowingNewGame)
}
```
It's just 5 lines of code, so it can't be hard to understand.  You can immediately see that this is going to return a `BorderPane` and that the `top` region is populated with a styled `Label` popped into an `HBox` so that it can be centred.  There's a bit of Kotlin magic here, the `HBox.centred()` extension function and the `Node.styleWith()` extension function, both of which we'll look at in a bit.  But for now, it's pretty clear what they do.

The `bottom` region gets an exit `Button` created through a builder/factory method and put in an `HBox` so that it too can be centred.

The `center` region gets a `StackPane` with two "boxes" created through builder methods and then modified with something called `showWhen`.  We'll look at `showWhen` later, but those builder methods now...

## The buttonBox() Method

``` kotlin
private fun buttonBox() =
		VBox(20.0, createButton("NEW") { isShowingNewGame.value = true }, createButton("LOAD") {}).centred()
```
This is technically one line of code, but it's been formatted out to 2 for readability.  It's a `VBox` with spacing of 20 pixels between the children.  The children are two `Buttons` created through that same builder/factory method we saw used in `createContent()`.  The contents of the `VBox` are centred.  That's all...pretty simple.

## The newGameBox() Method

``` kotlin
private fun newGameBox(): Region = BorderPane().apply {
	center = VBox(
			HBox(6.0, Label("Name:") styleWith "prompt-label", TextField(), Button("OK")).centred()
	).centred()
	top = Button("Back").apply { setOnAction { isShowingNewGame.value = false } } styleWith "transparent-button"
}
```
This is another `BorderPane` because we want to have the "Back" `Button` in the top left corner, and the `HBox` holding the input stuff centred.  There's no problem putting `BorderPane` inside another `BorderPane` or any other container class.  Also, there's nothing "heavyweight" about `BorderPane` that would have you want to avoid it.  

The `center` region has a `VBox` holding an `HBox` both of which are centred.  This puts the contents of the `HBox` right in the middle of the region.  Inside the `HBox` we have a styled `Label` that says, "Name:", a `TextField` and a `Button` labeled "OK".  Neither the `Button` nor the `TextField` do anything simply because they didn't in the original code.  

The "Back" `Button` has an action that sets the value of `isShowingNewGame` to false.  `isShowingNewGame` is the only field in the application now, and it's a `BooleanProperty`.  Clicking "Back" sets it to false.

## The Button Builder/Factory Method

``` kotlin
private fun createButton(title: String, action: EventHandler<ActionEvent>) = Button(title).apply {
		onAction = action
		styleWith("transparent-button")
}
```
This is an application of DRY to the original code.  All the `Buttons` were configured similarly, so this method does the repeated stuff.  Additionally, we have a parameter that accepts the `EventHandler` that's attached to the `Button` for its `OnAction Event`.  We're not setting the font or the background programmatically any more, either, now were using StyleSheets via `Node.styleWith()`.

## Kotlin Extension Methods

That's it for the layout code.  However, there are those mysterious Kotlin extension functions that were used all over the place.  Let's look at them:

``` kotlin
infix fun <T : Node> T.styleWith(selector: String): T = apply { styleClass += selector }
infix fun <T : Node> T.showWhen(observed: ObservableBooleanValue): T = apply {
    visibleProperty().bind(observed)
    managedProperty().bind(observed)
}

fun HBox.centred() = apply { alignment = Pos.CENTER }
fun VBox.centred() = apply { alignment = Pos.CENTER }
```
In Kotlin, you can add a method to an existing class without extending it using "Extension Functions".  These evaluated statically, so they're not really breaking any JVM rules, but they can save a lot of boilerplate with JavaFX.  You might also notice that these methods are declared outside of `Opening` and not part of any class at all.  This is something that you can do in Kotlin, and these are generally referred to as "top level functions".

Let's look at `HBox.centred()`.  It uses a "Scope Function" called `apply{}` which is defined as a member function of `Object` and passes the object it's called on to the body of `apply{}` as `this`.  Then the function returns the object itself.  Just like in Java, when `this` can be inferred, it can be left out.  In this case it shortens `this.setAlignment(Pos.CENTER)` to `alignment = Pos.CENTER` where `setAlignment()` is also replaced with the field accessor version.  

Unfortunately, `alignment` is defined at the `HBox` and `VBox` level, so we need an extension function for each one.  Each one will return the `HBox` or `VBox` it's called on, acting like a decorator.

For the `styleWith` there are a couple extra things.  First, `styleClass` is defined at `Node`, but we want our decorator to return the actual class of the object it's called on.  So, if we call `Label.styleWith()` we want a `Label` back, not a `Node`.  So we have to define it generically as a type `T` that's a subclass of `Node`.  The `styleClass` value is a `List` and we use the `List.plusAssign()` method to add a new selector to it, through `+=`.  

Finally, we have a modifier, `infix`.  Using this means that we can skip the `.` and the `()`.  You can define any function that has one passed parameter this way if you want.  This allows `Label("Hello") styleWith "some-selector"`.  Once again, it acts a decorator, returning the original object.

## The showWhen() Function

This is another infix decorator function, one that simply binds the `Visible` and `Managed Properties` of the `Node` to some `ObservableBooleanValue`.  

This is called in two ways, one with `isShowingNewGame` as the parameter, and one with `isShowingNewGame.not()` as the parameter.  In other words, each the two `Nodes` will only be visible when the other one is not.

Originally I had the calls to this method as part of the builder methods `buttonBox()` and `newGameBox()`, but then I thought it made more sense as part of the layout code since it was related more to how the entire layout functions.  Having the two calls sitting side-by-side in the code also made it easier to see how it was being used:

``` kotlin
center = StackPane(buttonBox() showWhen isShowingNewGame.not(), newGameBox() showWhen isShowingNewGame)
```

# How Does This Fix The Issues?

## Single Responsibility Principle

Now we don't have any methods that are larger than 5 lines long.  It's pretty hard to cram too much responsibility into 5 lines or less.  

One place where we do get close is with the definition of the `top` section of the main `BorderPane` and the `center` section of the `BorderPane` defined in `newGameBox()`.  In both these cases we're configuring multiple elements at one time.  In the second case it's a `VBox` holding an `HBox` holding a `Label`, `TextField` and `Button`.  

In practice I find that there's a certain "threshold of triviality" that can be applied to these situations.  Even if you're technically breaking SRP, are the second, third and fourth responsibilities so trivial, individually and collectively, that they can be left in with the primary responsibility?  The only trade-off with following SRP to the letter is that you need to drill down into the delegate methods to see what they do.  So it's never going to hurt much to split out and delegate the extra responsibilities, but you *might* lose some readability.

In the case of the `center` section defined in `newGameBox()`, it's only trivial because neither the `TextField` nor the `Button` do anything.  The moment that you start to add configuration in for these elements, it will breach the "threshold of triviality" and needs to be split out into its own method.

## Don't Repeat Yourself

Nothing is repeated here.  Between the extension functions and the redesign itself, all of the repetitious code has been excised from this application.

## YAGNI

In all honesty, the `TextField` and the "OK" `Button` feel a bit like YAGNI, simply because they don't do anything.  In truth, though, you are going to need them, and they're more like unfinished features than anything else.  That being said, half implemented features are almost as bad as YAGNI violations.  Features should be so small that there's never any possibility of implementing only half of one.

In this particular case, it seems like making that "OK" `Button` do something is going to require some application logic and access to a persistence layer.  That will mean adding a framework, which will mean some refactoring and redesign.  That should have been done before the `Button` was added to the screen.

## Excessive Coupling

The entire application now has only **one** variable defined, and that is the final `BooleanProperty` field, `isShowingNewGame`.  It is very, very difficult to have coupling without variables to reference.  In fact, `isShowingNewGame` was only introduced to act as a central coupling point for the functionality of the `Buttons` and layout swapping - eliminating the need to instantiate those `Buttons` or layouts as variables in order to reference them from other layout elements.  

## Bad Names

Almost no variables means almost no variables to name, too!  What we do have, however, are methods that need good names.  I feel like the extension functions, `styleWith()`, `showWhen()` and `centred()` have good names, although perhaps `styledWith()` and `shownWhen()` - changing the tense, might be better.  

Maybe `createButton()` might be more descriptive as `createBigButton()` or `createMainButton()`, but with an application this small it hardly matters.  If the application grew and more `Button` types were added, then you'd definitely need to change this name.

The same goes for the other methods.  Perhaps `buttonBox()` is fine for now, but maybe `menuBox()` would be more descriptive if the application grows.  The main method, `createContent()` will become `build()` if this is changed over to a framework, and this code is all moved into a `Builder<Region>`.

## Changing Layouts

This layout is now static, but it behaves dynamically in response to the value in `isShowingNewGame`.  There no longer is any attempt to change the running layout.

## Understandable Layout

It's 20 lines of code.  It's a `BorderPane` and you can see at a glance that only the `top`, `center` and `bottom` sections are used.  You can see right away that the centre is a `StackPane` with two layouts that alternate in visibility.  You can drill down into those two layouts and understand them easily.  


# Conclusion

It's pretty easy to look at the final code and say, "Yeah, but that's a pretty trivial example.  It's only 20 lines of layout code.  You can't do that with something complex."  But yet...the original code was over 100 lines long, and was non-trivial enough that the author got stuck trying to figure out how to make it work.  

Hopefully, you can also see how Kotlin makes life so much better with JavaFX.

One really big rule when writing layout code is *NOT* to reference `Nodes` from other `Nodes`, especially when those `Nodes` aren't closely related in the layout.  Especially, especially in the case where they are so unrelated in the layout that you need to instantiate one of them as a field, so that it's global to the layout.  Instead, create a `Property` as a field, or in your Model, and then reference that `Property` from the `Nodes` that need to interact (but now indirectly).
