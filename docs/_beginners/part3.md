---
title: Part 3 - User Interaction
short_title: Part 1
permalink: /beginners/part3
screenshot_1: /assets/beginners/part3_1.png
---

# What You'll Learn

# Adding User Interaction

## The Code So Far

``` java
public class Main extends Application {

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        Scene scene = new Scene(createContents(), 400, 200);
        primaryStage.setScene(scene);
        primaryStage.show();
    }

    private Region createContents() {
        HBox results = new HBox(new Label("Name:"), new TextField(""));
        results.setSpacing(6);
        results.setPadding(new Insets(0,0,0,50));
        results.setAlignment(Pos.CENTER_LEFT);
        return results;
    }
}
```

# Adding the Output and a Button

Now we are going to add some interaction to the application.  We'll add a `Button` and put the output `Label` back on the screen.  We'll set up the `Button` so that when it's clicked, it loads "Hello " plus the name into the output `Label`.

The code now looks like this:

``` java
public class Main extends Application {

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        Scene scene = new Scene(createContents(), 400, 200);
        primaryStage.setScene(scene);
        primaryStage.show();
    }

    private Region createContents() {
        TextField inputTextField = new TextField("");
        HBox inputRow = new HBox(new Label("Name:"), inputTextField);
        inputRow.setSpacing(6);
        inputRow.setAlignment(Pos.CENTER);
        Label outputLabel = new Label("");
        Button actionButton = new Button("Hello");
        actionButton.setOnAction(evt -> outputLabel.setText("Hello " + inputTextField.getText()));
        VBox results = new VBox(20, inputRow, outputLabel, actionButton);
        results.setAlignment(Pos.CENTER);
        return results;
    }
}
```

I've wrapped the entire content in a `VBox`, which presents the contained `Nodes` vertically stacked on the screen.  Then I've added the `Label` for the output (called `outputLabel`), and the `Button` to trigger the work (called `actionButton`).
An `EventHandler` was added to the `actionButton` to take an `ActionEvent` which will trigger an update on the contents of the `outputLabel`.


And the output looks almost like this:

![Screenshot]({{page.screenshot_1}})

I've put a green border around the outer `VBox` and the red border remains around the `HBox`.

## There's a Lot Wrong with this code

Let's take a look at the problems with this code:

### It's Getting Busy

This code no longer follows the "Single Responsibility Principle".  It does the following:

- Populates the input row and configures its layout
- Creates the output `Label`
- Creates the `Button`
- Defines the code which updates the output `Label`



### There's Way Too Much Coupling

#### What's Coupling, and Why is it Bad?

Coupling is when components of your application rely on the implementations of other components of your application.  This results in a situation where two or more components are dependent on the structure of each other, making it difficult to modify one component without needing to modify another component.  And, if that second, dependent, component has other components that are dependent on it... You see where this goes.

Coupling is the thing that you need to fight every step of the way when you're building an application because it sneaks in almost every time you stop looking for it.  So get into the habit of looking for coupling and avoiding it constantly.

#### Where's the Coupling Here?

`actionButton` calls `inputTextField.getText()`
: This is a big problem.  These two elements are now totally coupled.  Let's say that `inputTextField` is changed to some other type of Node that doesn't have a `getText()` method.  Maybe the application is intended to be used in a situation where there are a limited number of users, and the name is picked using a `ComboBox`, which doesn't have a `getText()` method.  Then you'll need to update the `Button` as well.

`actionButton` calls `outputLabel.setText()`
: This is the same type of coupling as above.

`actionButton` needs to access both `outputLabel` and `inputTextField`
: These elements are coupled in the code.  Any attempt to move the instantiation of either of these two `Nodes` into another method is going to cause trouble with `actionButton`.

The update of `outputLabel` happens inside the `EventHandler` definition.
: The logic about updating the contents of the `Label` are now encapsulated inside the `EventHandler`

## Fixing the Coupling and Holding to the Single Responsibility Principle

Let's look at some code that will do this:

``` java
public class Main extends Application {

    public static void main(String[] args) {
        launch(args);
    }

    private final StringProperty greeting = new SimpleStringProperty("");
    private final StringProperty name = new SimpleStringProperty("");

    @Override
    public void start(Stage primaryStage) {
        Scene scene = new Scene(createContents(), 400, 200);
        primaryStage.setScene(scene);
        primaryStage.show();
    }

    private Region createContents() {
        VBox results = new VBox(20, createInputRow(), createOutputLabel(), createGreetingButton());
        results.setAlignment(Pos.CENTER);
        return results;
    }

    private Node createGreetingButton() {
        Button results = new Button("Hello");
        results.setOnAction(evt -> setGreeting());
        return results;
    }

    private Node createInputRow() {
        TextField textField = new TextField("");
        textField.textProperty().bindBidirectional(name);
        HBox hBox = new HBox(6, new Label("Name:"), textField);
        hBox.setAlignment(Pos.CENTER);
        return hBox;
    }

    private Node createOutputLabel() {
        Label results = new Label("");
        results.textProperty().bind(greeting);
        return results;
    }

    private void setGreeting() {
        greeting.set("Hello " + name.get());
    }
}
```

### StringProperty???

The first thing you'll notice is that we've added two `StringProperty's` fields to the class.  Together these two `Properties` are going to act as our "State" for the application.  One of them, the `name` will hold whatever the user types into the `TextField` and the other will hold the greeting that will display in the output `Label`.

Putting these two `Properties` into the class allows all of the other coupling to be removed.  

### `createContents()` Now Just Sets Up the Main VBox


### All of the various `Nodes` Now Have Generic Names

Since the layout of the screen has now been split into tiny, clearly named methods, there's no value in having meaningful variable names for the individual `Nodes`.  The scope of each of these variables is so small now that there's no added clarity in having complicated names for them, and simple names like `hBox` should indicate to any reader that these have a limited scope.

### All of the Builder Methods Return Node or Region



### The EventHandler in the Button is Just a Trigger to call setGreeting()
