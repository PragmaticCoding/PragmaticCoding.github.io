---
title:  "JavaFX Anti-Pattern: The Dispatch EventHandler"
date:   2021-04-19 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/dispatchEventHandler
excerpt: Stop treating your JavaFX controls as data and passing them global EventHandlers.  
---

Many programmers look for on-line tutorials and examples to find out how to code common JavaFX structures and techniques.  Unfortunately, not all the code you'll find on the web is high quality and there are a number of bad techniques that have been replicated over and over again on numerous sites.  One of these is the structure that I call the "Dispatch EventHandler".

### A Real Example From the Web

There was an answer to a question on [StackOverflow](https://stackoverflow.com/a/67134797/36223)  recently about adding actions to Buttons in JavaFX.  

The solution was posted is in the listing below.  To be fair, the person posting the answer was just solving the problem as asked, and kept the same overall approach that the person who posted the question had taken.  I've used the code from the answer instead of from the question because this code actually runs and produces a partially correct result:

``` java
public class LatinTranslator extends Application {
    private Label myLabel;

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        myLabel = new Label();

        Button leftButton = new Button("Sinister");
        Button centerButton = new Button("Medium");
        Button rightButton = new Button("Dexter");

        centerButton.setAlignment(Pos.CENTER);

        leftButton.setOnAction(new ButttonClickHandler());
        centerButton.setOnAction(new ButttonClickHandler());
        rightButton.setOnAction(new ButttonClickHandler());

        VBox Vbox = new VBox(20, myLabel, leftButton, centerButton, rightButton);

        Scene scenebox = new Scene(Vbox, 300, 500);
        Vbox.setAlignment(Pos.CENTER);

        primaryStage.setScene(scenebox);
        primaryStage.setTitle("Latin Translator");

        primaryStage.show();
    }

    class ButttonClickHandler implements EventHandler<ActionEvent> {

        @Override
        public void handle(ActionEvent event) {
            Object source = event.getSource();
            if (source instanceof Button) {
                Button button = (Button) source;
                String text = button.getText();
                if ("Sinister".equals(text)) {
                    myLabel.setText("left");
                }
            }
        }
    }
}
```

I've seen this approach a lot.  I wrote code like this when I started out with JavaFX.

This is probably copypasta from the early days of JavaFX 2, most likely starting with a tutorial on creating a menu.  The same type of structure was used, with the menu text being used to decide which action to take in the `ButtonClickHandler` code.  It's possible that this was a hold-over from [Swing](https://www.javatpoint.com/java-jmenuitem-and-jmenu), where this approach was standard for building JMenu's.

Searching for examples today, I couldn't find any online tutorials for JavaFX menus that used this technique, which is good.  However, the fact that it would pop up in a recent StackOverflow question posted by someone who was probably new to JavaFX seems to indicate that there are examples out there for beginners to find.


### The Problem

It's the `ButtonClickHandler` class that is the problem.  It's a "Dispatch EventHandler".  The idea is that you create a single `EventHandler` that will called by a number of controls, and it will somehow figure out which one called it, and what it should do for each one.  

It's a very round-about way to program this.  It's worse than going around three sides of a square - more like going around seven sides of an octagon - and it introduces a level of complexity that simply doesn't serve any purpose at all.  The person writing the answer in this example didn't even bother to put in the actions for two out of the three buttons; probably because it was too much work and didn't contribute to the solution.

Using this approach treats the Buttons like data, and ignores the fact that Button is a Control with the ability to perform actions on its own.  This, even though the program invokes the Buttons' `OnAction()` method!

Another big issue with this code is that it tightly couples the display elements to the action elements.  What happens if you discover that you've spelled one of the Latin words on the buttons incorrectly?  Now you have to correct both the button and the action code.  Furthermore, if you're coming at this as a maintenance programmer to fix the spelling error, you need to understand that the button text isn't just display but is actually used as data to control the logic flow somewhere else.  That's an easy thing to miss!

### A Better Approach

The program could be much simpler:

``` java
public class Translator extends Application {

    private final Label myLabel = new Label();

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        VBox vBox = new VBox(20,
                              myLabel,
                              createButton("Sinister", "Left"),
                              createButton("Medium", "Middle"),
                              createButton("Dexter", "Right"));
        vBox.setAlignment(Pos.CENTER);
        primaryStage.setScene(new Scene(vBox, 300, 500));
        primaryStage.setTitle("Latin Translator");
        primaryStage.show();
    }

    private Node createButton(String buttonText, String labelText) {
        Button button = new Button(buttonText);
        button.setOnAction(evt -> myLabel.setText(labelText));
        return button;
    }
}
```
First of all, this new version is much smaller.  There are roughly half as many lines of code compared to the original version - especially considering  that this version handles the translations for all of the Buttons, not just the first one.  Generally speaking, less code is going to be easier to read and to understand; and that certainly is the case here.

I've also stripped out some unneeded variables and their initialization to streamline and simplify the code.  This contributes to the reduction in lines of code.

I've put the actual instantiation of the Buttons into its own method here for a couple of reasons:  

- First, the code in that method would otherwise be repeated over three times, and you'd need to create a variable for each one.  If you had more than three buttons, the situation would just get worse.  

- Secondly, note that the method returns "Node", not button.  This is an important distinction because it really emphasizes that to the layout it's just a generic screen element.  Everything that makes it different and unique is contained within it, and the layout code has no need to even know that this is a button with an action.  The nature of the button is now completely decoupled from the layout of the GUI, as it should be.

### There are Even More Benefits

Given that the context for this code seems to be an application for showing translations of words from one language to another, it seems unlikely that you'd want to hand code each word and translation on screen after screen after screen.  In that case, you'd probably have some database of translations, and you'd pass some subset of your database to the screen for display.

To simulate that, we'll add a method that will create a `Map<String, String>` of words and translation to provide our database results.  Then we'll stream that Map and turn it into a list of Buttons that can be put in the VBox:

``` java
public class Translator extends Application {

    private final Label myLabel = new Label();

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        VBox vBox = new VBox(20, myLabel);
        vBox.getChildren()
            .addAll(initializeData().entrySet()
                                    .stream()
                                    .map(entry -> createButton(entry.getKey(), entry.getValue()))
                                    .collect(Collectors.toList()));
        vBox.setAlignment(Pos.CENTER);
        primaryStage.setScene(new Scene(vBox, 300, 500));
        primaryStage.setTitle("Latin Translator");
        primaryStage.show();
    }

    private Node createButton(String buttonText, String labelText) {
        Button button = new Button(buttonText);
        button.setOnAction(evt -> myLabel.setText(labelText));
        return button;
    }

    private Map<String, String> initializeData() {
        Map<String, String> results = new HashMap<>();
        results.put("Sinister", "Left");
        results.put("Medium", "Middle");
        results.put("Dexter", "Right");
        return results;
    }
}
```

Philosophically, what's important here is that the key-value pairs are turned into independent Button controls work on their own.  These can then be placed anywhere on the scene without requiring any modification of anything else.  

You can also see how the `createButton()` method does not need any modification to work as a translation routine to turn each key-value pair into a JavaFX Node.  

### One Last Fix

There's one dependency on the screen that is still hanging around, and that's the direct update of `myLabel.setText()`.  This is forcing the `myLabel` element to be a field in the class.  It also forces each Button to be aware of the presence of the `myLabel` screen element, which is more needless coupling at the GUI layout level.

This can be fixed by creating a StringProperty to hold the translation, and then binding the Text property of `myLabel` to that StringProperty.  This may seem like nit-picking, but if you were building this screen with an MVC structure, then both the `Map` of words and translations and the translation for the word associated with a clicked button would be defined in the Model.  This would meant that the buttons and the translation label are coupled to the Model, as they should be, and not to each other:

``` java
public class Translator extends Application {

    private final StringProperty translation = new SimpleStringProperty("");

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        Label myLabel = new Label();
        myLabel.textProperty().bind(translation);
        VBox vBox = new VBox(20, myLabel);
        vBox.getChildren()
            .addAll(initializeData().entrySet()
                                    .stream()
                                    .map(entry -> createButton(entry.getKey(), entry.getValue()))
                                    .collect(Collectors.toList()));
        vBox.setAlignment(Pos.CENTER);
        primaryStage.setScene(new Scene(vBox, 300, 500));
        primaryStage.setTitle("Latin Translator");
        primaryStage.show();
    }

    private Node createButton(String buttonText, String translationText) {
        Button button = new Button(buttonText);
        button.setOnAction(evt -> translation.set(translationText));
        return button;
    }

    private Map<String, String> initializeData() {
        Map<String, String> results = new HashMap<>();
        results.put("Sinister", "Left");
        results.put("Medium", "Middle");
        results.put("Dexter", "Right");
        return results;
    }
}
```

And that's all.  This has gone from something clumsy enough that the original author couldn't be bothered to complete the coding for all of the Buttons in the sample to something so simple that it was trivial to simulate how it would work integrated with a database.  Most importantly, by making the Buttons stand-alone controls, the layout of the screen is now completely decoupled from the actions of the screen.  Those Buttons don't even have to be Buttons anymore, and `myLabel` could change to some other kind of Node.
