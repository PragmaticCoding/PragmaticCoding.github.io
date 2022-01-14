---
title:  "TextFields and TextFormatter - Part 1"
date:   2021-05-11 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/textformatter1
excerpt: JavaFX contains powerful tools to turn simple TextFields into specialized entry fields for any kind of data you can think of.
---
JavaFX contains powerful tools to turn simple `TextFields` into specialized entry fields for any kind of data you can think of.

### Introducing TextFormatter

If you are writing a JavaFX application that needs a user to type in information, then you'll almost certainly need to use the TextField control.  But what if you need to make sure that the user enters valid data?  What if the data you need is not a string?  How do you convert it to the type that you need?

Generally speaking, there are three approaches to handling verification of user data:

   1.  Check the data in all of the fields whenever the data is saved/acted on.   
   1.  Flag fields with invalid data and disable the save/action triggers.
   1.  Prevent the user from entering invalid data.


The first approach usually involves creating some kind of messaging system with a way to tell the user what's wrong with their input after hitting the save button.  It can be disruptive, and requires building a flow to handle the error messaging.

The second approach is most useful when there are interactions between the data entry elements and what is considered a valid entry can change based on other controls on the screen.  For instance, if there are an upper and lower limit to a numeric value that can change based on user selections.  In such a case, if the user changes one of those other controls, a value previously entered in a field and considered valid at the time may no longer be valid.  Libraries such as [ControlsFX](https://github.com/controlsfx/controlsfx) have classes which can do this nicely.

The third case is what we're going to talk about here:  Setting up a `TextField` so that the user can only enter data which fits a particular structure when they are editing it.  It also has the added benefit that it will handle conversions between types for you.  `TextFields` are, by definition, controls to enter text, But by attaching a `TextFormatter` to your `TextField` it can automatically convert your text input into, for instance, an integer, allowing you to associate your `TextField` with integer data.

Installing a `TextFormatter` into a `TextField` is trivial, just call the `TextField.setTextFormatter()` method.  The `TextFormatter` class itself, however, needs to be set up properly to handle the work.  `TextFormatter` has two main parts; a filter which handles user inputs, and a converter which maintains synchronicity between the text in the `TextField` and a value property which is some other data type.  

### A Simple Example - A Whole Number TextField

Let's start by looking at a straight forward use of `TextFormatter` - creating a TextField that will only accept a string of numbers that will be converted into positive integer.

In order to test the code, we'll need a basic screen with some controls:
``` java
public class Main extends Application {

    private ObjectProperty<Integer> valueProperty = new SimpleObjectProperty<>(0);

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        Scene scene = new Scene(new TestPane(), 300, 100);
        primaryStage.setScene(scene);
        primaryStage.show();
    }

    public class TestPane extends BorderPane {
        public TestPane() {
            TextField textField = new TextField();
            setCenter(new VBox(10,
               new HBox(6, new Text("TextField 1"), textField),
               new HBox(6, new Text("TextField 2"), new TextField())));
        }
    }
}
```
This has two extra items in it beyond the TextField to test the `TextFormatter`:  An integer property that we'll use later to test the conversion (think of this as the "model" in an MVC setup), and a second TextField just so that we can see what happens when focus is gained or lost in the test TextField.  When you run this, you'll get a window with a couple of labels and `TextFields`, and the data entry in both `TextFields` is free form and will accept anything.

#### Adding a Converter to the TextField

Now let's add a converter to the TextField:

``` java
public class Main extends Application {

    private ObjectProperty<Integer> valueProperty = new SimpleObjectProperty<>(0);

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        Scene scene = new Scene(new TestPane(), 300, 100);
        valueProperty.addListener(((observable, oldValue, newValue) -> {
            System.out.println("Value changed -> Old Value: " + oldValue + ", New Value: " + newValue);
        }));
        primaryStage.setScene(scene);
        primaryStage.show();
    }

    public class TestPane extends BorderPane {
        public TestPane() {
            TextField textField = new TextField();
            TextFormatter<Integer> textFormatter = new TextFormatter(new IntegerStringConverter());
            textFormatter.valueProperty().bindBidirectional(valueProperty);
            textField.setTextFormatter(textFormatter);
            setCenter(new VBox(10,
              new HBox(6, new Text("TextField 1"), textField),  
              new HBox(6, new Text("TextField 2"), new TextField())));
        }
    }
}
```
Here we've added a `TextFormatter` with just a converter applied to it.  This `TextFormatter` wraps an Integer property, so now we can bind that property to the already existing `valueProperty` field.  From the perspective of externally accessing the value in the TextField, we aren't really interested in the `TextField's`, `textPropery` any more, all the access should be done through the `TextFormatter`.  So that it's easier to see what's going on with the converter, and when it fires, there's a now listener on the `valueProperty` field which will display how it's changing on the console.

The converter here is an `IntegerStringConverter`, which extends `StringConverter<Integer>` and is a standard part of JavaFX.  `StringConverter` has just two methods, `fromString()` and `toString()`.

If you run this code, you'll see that it allows you to type in anything you like, but if you trigger the converter by hitting \<Enter\> or moving the focus away from the `TextField`, it will reset your value back to the last valid value if the conversion fails.  So if you start with "0", and then replace it with "abcd" and then hit \<Tab\>.  It will put the value back to "0".

The one thing `IntegerStringConverter` won't do is prevent the user from entering negative numbers.  For this, we'll need to create a custom converter that only accepts positive integers and ensures that negative values cannot be entered:

``` java
public class PositiveIntegerStringConverter extends IntegerStringConverter {

    @Override
    public Integer fromString(String value) {
        int result = super.fromString(value);
        if (result < 0) {
            throw new RuntimeException("Negative number");
        }
        return result;
    }

    @Override
    public String toString(Integer value) {
        if (value < 0) {
            return "0";
        }
        return super.toString(value);
    }

}
```

You can install this new converter when the `TextFormatter` is instantiated.  Call the constructor like this: `new TextFormatter(new PositiveIntegerStringConverter())`.

If you look at the source code for `IntegerStringConverter`, you'll see that the `fromString()` method doesn't do anything special to check for non-numeric characters in the string value.  It does some checking for empty or Null strings, and then just returns using, `Integer.valueOf(value)`.  As I'm sure you know, calling `Integer.valueOf(value)` on a string with non-numerics will throw an exception and `IntegerStringConverter` does nothing to prevent this from happening.  The `TextFormatter` has a try/catch block that resets the string value if the new string fails to convert.  To be consistent with this, the new converter explicitly throws a `RuntimeException` if the new, converted value is negative.  This seems ugly, but since the `fromString()` method is only ever called in response to changes in the `TextField` to which it is attached, it's probably okay.

The reverse direction is a little more problematic.  The `toString()` method is only going to be called from external changes to the `Value` property of the `TextFormatter`.  So throwing a `RuntimeException` is probably going to cause issues.  Returning `Null` is also a problem, as it doesn't trigger an update in the bound View Model property.  Setting it to "0", causes the change to the bound property when the field loses focus.

#### Controlling the Input with a Filter

At this point, the code just about meets the minimum criteria to work.  It won't let you put in invalid data, and it provides a value of the correct type for the View Model.  From a user experience perspective, this is still less than optimal, as it silently changes the user's input without any warning if they enter invalid data.  That's never a good thing.

The way to handle this is to add a filter to the `TextFormatter`.  A filter is a class that accepts the users change to the string in the TextField, and ensures that it conforms to a set of rules.  The "change" is sent through the filter in the form of an object of type `TextFormatter`.Change.  The filter can reject the change, let it go through, or it can modify it in some way so that it changes the way that the TextField behaves.

``` java
public class PositiveIntegerFilter implements UnaryOperator<TextFormatter.Change> {

    @Override
    public TextFormatter.Change apply(TextFormatter.Change change) {
        if (change.getControlNewText().matches("[0-9]*")) {
            return change;
        }
        return null;
    }
}
```
A `UnaryOperator` is just a specialized case of the Function interface which returns data of the same type that it accepts.  As the filter, it accepts `TextFormatter.Change`, and returns `TextFormatter.Change`.

This filter is very simple, it just checks to make sure that the new string after the change is applied will only contain digits.  It does this by checking return value of `getControlNewText()`, which is what the string in the `TextField` would be if this change was applied, and then letting the change go through if it meets the regex test.  Otherwise, the filter returns Null, and the change is blocked.

You can apply this filter by changing the instantiation of the `TextFormatter` by calling `new TextFormatter(new PositiveIntegerStringConverter(), 0, new PositiveIntegerFilter())`.  When you do this, the `TextField` will now allow you to type in numbers, but anything else, including "-" and ".", will be swallowed up by the filter.

At this point, the `TextFormatter` is complete and fully functional.  Technically, we could probably go back and remove the negative checking in the `fromString()` function of the converter, but it's probably best to leave it in so that `PostiveIntegerStringConverter` really does what its name implies.  The important thing is that the filter and the converter need to be compatible with each other.  It is possible to set up a converter with at `toString()` method that generates a string which cannot be edited at all by the rules enforced by the filter.

In [Part 2](https://pragmatic-coding.hashnode.dev/javafx-textfield-and-textformatter-part-2), we'll take a deeper look at TextFormatter.Change, and how to create a more complicated filter that makes large differences to the way that TextField works.
