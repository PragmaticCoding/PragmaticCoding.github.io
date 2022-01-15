---
title:  "TextFields and TextFormatter - Part 2"
date:   2021-05-11 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/textformatter2
excerpt: In Part 2, we'll build a filter and converter which will handle decimal data input with a fixed number of decimal places.
---

In [Part 1](/javafx/textformatter1) of this series, we looked at how to use `TextFormatter` to customize the behaviour of a TextField to handle specialized data entry.  In this part we'll build a filter and converter which will handle decimal data input with a fixed number of decimal places.

## The Objective

The best way to see how `TextFormatter` works is to walk through the creation of a `TextField` customized to accept only a certain kind of input.  For this example, we'll build a decimal input field with a fixed number of decimal places.  To make it interesting, we'll stipulate the following requirements:

- Only a set amount of decimal places are allowed
- If the data entry would result in too many decimal places then some will be trimmed out:
    - If the data entry is at the very end of the number, then the first decimal place will be removed
    - If the data entry is not at the very end of the number, then the last decimal place will be removed
- If the data entry would result in too few decimal places, a "0" will be appended to the end of the number
- When the TextField gains focus, the whole number portion of the field will be selected.
- When the decimal point key is pressed:
    - If the cursor is currently in the whole number portion, then the decimal portion will be selected.
    - If the cursor is currently in the decimal portion, then the whole number portion will be selected.
- When the "-" key is pressed, no matter where the cursor is:
    -  If the number is currently negative, then the "-" will be removed from the beginning of the number and the cursor will remain where it is.
    - If the number is currently positive, then a "-" will be placed at the beginning of the number, and the cursor will remain in the same position.

In other words, this will functionally split the contents of the `TextField` into two parts with the `<.>` key used to toggle between them.  The `<->` key can be pressed when the cursor is in any place in either portion and it will toggle a negative sign at the beginning of the number.  Any data entry that would cause the number to have the wrong number of decimal places will be automatically corrected by scrolling off one of the digits, or appending a "0" to the number.

### Is this a good way to design a fixed place decimal field?  

I'm not sure.  

On one hand, it's highly functional and works smoothly.  If someone just types "1234.56" when there are two decimal places allowed, it will result in "1234.56", as expected.  

The rest of the actions, however, are a little counter-intuitive and would require explanation or some exploration on the part of the user.  That's probably not the best design, especially if it's deployed in an application with a lot of single-time users.

Any UI design that requires explanation or training to use is probably not awesome.
{: .notice--warning}

All that aside, it is a good example to explore how you can use `TextFormatter` to create a much more sophisticated input control out of a simple TextField.

##  The Converter

We'll start with the converter since it's the easiest part:

``` java
public class FixedDecimalConverter extends DoubleStringConverter {

    private final int decimalPlaces;

    public FixedDecimalConverter(int decimalPlaces) {
        this.decimalPlaces = decimalPlaces;
    }

    @Override
    public String toString(Double value) {
        return String.format("%." + decimalPlaces + "f", value);
    }

    @Override
    public Double fromString(String valueString) {
        if (valueString.isEmpty()) {
            return 0d;
        }
        return super.fromString(valueString);
    }

}
```

The two things that make this different from the `DoubleStringConverter` is that it always forces the `toString()` result to have the correct number of decimal places, and that it turns an empty input string into a zero in the `fromString()` method.

## The Filter

The Filter is an implementation of `UnaryOperator<TextFormatter.Change>`, which is used to validate and modify any change that the user makes in the TextField.  `UnaryOperator` is just a `Function` which returns the same data type as it takes.  So in the case of the Filter, it takes `TextFormatter.Change` as the input value, and returns a `TextFormatter`.Change as its result.

### The Structure of TextFormatter.Change

You can think of `TextFormatter.Change` as clean way to capture each key presses and mouse actions in the TextField, with JavaFX itself handling all of the troublesome details about capturing and applying the keystrokes.  

Using TextFormatter.Change means you don't need to fuss about capturing KeyEvents.
{: .notice--info}

The `Filter` is a step between capturing the user action and applying it, and that allows us to write logic that can not just filter out actions, but change them and the way that they impact the `TextField`.

`TextFormatter.Change` contains a lot more information than just the keystroke itself.  Here are some of the essential components of `TextFormatter.Change`:

- The "Change" Part of the Change:

    These are the elements of the change itself.  All of these can be manipulated to alter the impact of the change:
    - The text of the change.
    - The range that the change applies to in the TextField contents
    - The new positions of the anchor and caret

- The current `TextField` value:

    These are accessed through methods that are called, `getControl...()`.  `TextFormatter.Change` refers to the `TextField` as the "Control", because it can be used with more than just `TextField`.
    - The current text in the `TextField`
    - The current position of the caret and the anchor

- The new `TextField` value.  This is access through the `getControlNewText()` method.

    You can see what the impact of the change will be on the text value held in the `TextField`.

There is one caveat when manipulating the change:

The value returned by getControlNewText() is updated immediately when the change has been modified.  
{: .notice--warning}

So if you need to reference the impact of the original change after you've modified it, you'll need to make a copy before you start.

### Filtering the Change

The first thing to do is to ensure that the results of the change are a valid decimal string, so we'll use regex to check that:

``` java
public class FixedDecimalFilter implements UnaryOperator<TextFormatter.Change> {

    @Override
    public TextFormatter.Change apply(TextFormatter.Change change) {
        if (change.getControlNewText().matches("-?([0-9]*)?(\\.[0-9]*)?")) {
            return change;
        }
        return null;
    }
}
```
Here you can see we've created a new class which implements `UnaryOperator`, and then supplies the logic for the `apply()` method.  The code in there now will act as the backstop to the filter, making sure that whatever the final version of the change looks like, it won't violate the requirement to have a decimal string.  This code will allow any number of decimal places, but we'll see how the rest of the controls in the Filter will handle this nicely.

So that we can test it, here's a "Main" class which will launch a window with some `TextFields`:

``` java
public class FixedDecimalMain extends Application {

    private ObjectProperty<Double> valueProperty = new SimpleObjectProperty<>(0d);

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        Scene scene = new Scene(new FixedDecimalMain.TestPane(), 300, 100);
        valueProperty.addListener(((observable, oldValue, newValue) -> {
            System.out.println("Value changed -> Old Value: " + oldValue + ", New Value: " + newValue);
        }));
        primaryStage.setScene(scene);
        primaryStage.show();
    }

    public class TestPane extends BorderPane {
        public TestPane() {
            TextField textField = new TextField();
            TextFormatter<Double> textFormatter = new TextFormatter(new FixedDecimalConverter(2), 78, new FixedDecimalFilter());
            textFormatter.valueProperty().bindBidirectional(valueProperty);
            textField.setTextFormatter(textFormatter);
            setCenter(new VBox(10, new HBox(6, new Text("TextField 1"), textField), new HBox(6, new Text("TextField 2"), new TextField())));
        }
    }
}
```
Now, if you run this you'll see that you can only enter characters which result in a valid decimal string.  Otherwise, your keystrokes are swallowed up by the `Filter`.

### Selecting the Whole Number Part on Focus

The first thing you'll notice if you run this is that the entire contents of the `TextField` are selected when the control gains focus.  This isn't what are requirements state, only the whole number part of the string should be selected.  How to do this?

In order to investigate this, let's add some console output to monitor what's coming into the `Filter`:

``` java
public class FixedDecimalFilter implements UnaryOperator<TextFormatter.Change> {

    @Override
    public TextFormatter.Change apply(TextFormatter.Change change) {
        System.out.print("Change: >" + change.getText() + "< Range: [" + change.getRangeStart() + ", " + change.getRangeEnd() + "]");
        System.out.print(" Selection: [" + change.getSelection().getStart() + ", " + change.getSelection().getEnd() + "]");
        System.out.println(" Anchor: " + change.getAnchor() + " Caret: " + change.getCaretPosition());
        System.out.println("    TextField: >" + change.getControlText() + "<, L: " + change.getControlText().length());
        if (change.getControlNewText().matches("-?([0-9]*)?(\\.[0-9]*)?")) {
            return change;
        }
        return null;
    }
}
```
No when you launch the application, you get the following console output:

```
Change: >< Range: [0, 0] Selection: [0, 4] Anchor: 4 Caret: 0
    TextField: >0.00<, L: 4
```

And if you hit `Tab` twice, to toggle the focus to the second TextField and back, you'll end up with the following output:

```
Change: >< Range: [0, 0] Selection: [0, 4] Anchor: 4 Caret: 0
    TextField: >0.00<, L: 4
Change: >< Range: [0, 0] Selection: [0, 0] Anchor: 0 Caret: 0
    TextField: >0.00<, L: 4
Change: >< Range: [0, 0] Selection: [0, 4] Anchor: 4 Caret: 0
    TextField: >0.00<, L: 4
```

So it appears that when focus is gained on the `TextField`, it generates a `Change` with empty text and with the anchor set at the end of the string and the caret at the beginning.  When focus is lost, it sends another `Change` with empty text and both the anchor and the caret set to the beginning of the string.

It's also possible to select the contents of a `TextField` by double-clicking on it.  This is the output that generates:

```
Change: >< Range: [0, 0] Selection: [0, 0] Anchor: 0 Caret: 0
    TextField: >0.00<, L: 4
Change: >< Range: [4, 4] Selection: [0, 4] Anchor: 0 Caret: 4
    TextField: >0.00<, L: 4
```
The first line is when it responds to the first click, and the second when it processes it as a double click.  It's almost the same as `Tab`, but the caret and the anchor are reversed.

It looks like the best way to detect this is to check for a `Change` with empty text and a range which includes the entire contents of the `TextField`.  Let's put some code in the filter to catch that and change it to select just the whole number portion:

``` java
public class FixedDecimalFilter implements UnaryOperator<TextFormatter.Change> {

    @Override
    public TextFormatter.Change apply(TextFormatter.Change change) {
        System.out.print("Change: >" + change.getText() + "< Range: [" + change.getRangeStart() + ", " + change.getRangeEnd() + "]");
        System.out.print(" Selection: [" + change.getSelection().getStart() + ", " + change.getSelection().getEnd() + "]");
        System.out.println(" Anchor: " + change.getAnchor() + " Caret: " + change.getCaretPosition());
        System.out.println("    TextField: >" + change.getControlText() + "<, L: " + change.getControlText().length());

        if (change.getText().isEmpty() && isEverythingSelected(change)) {
            change.selectRange(0, change.getControlText().indexOf("."));
            return change;
        }
        if (change.getControlNewText().matches("-?([0-9]*)?(\\.[0-9]*)?")) {
            return change;
        }
        return null;
    }

    private boolean isEverythingSelected(TextFormatter.Change change) {
        return (change.getSelection().getStart() == 0) && (change.getSelection().getEnd() == change.getControlText().length());
    }
}
```
This is our first example of manipulating the `Change` to alter the behaviour of the `TextField`.  If any action attempts to select the entire string, it will intercept it and select the whole number portion instead.  Also, since this can never alter the `TextField` string, there's no point in running it through the regex to see if the `Change` is valid, so we return the altered `Change` immediately.

### Handling the "." Key

Now we want the "." key to toggle the input between the whole number and decimal portions of the string, selecting the entire part.  So we'll need to capture the decimal point `Change` and manipulate it.  Here's the logic that will do that:
``` java
int decimalPos = change.getControlText().indexOf(".");
if (change.getText().equals(".")) {
    change.setText("");
    change.setRange(0, 0);
    if (change.getControlCaretPosition() <= decimalPos) {
        change.setCaretPosition(decimalPos + 1);
        change.setAnchor(change.getControlText().length());
    } else {
        change.setCaretPosition(decimalPos);
        change.setAnchor(0);
    }
    return change;
}
```
We've introduced `decimalPos` because it we're going to be using it in two places now.

First, we clear the text in the `Change`, since we don't actually want to add a "." to the string.  Then we set the `Change` range to (0,0) since we don't want to be removing any text either.  So now the `Change` can't actually change the `TextField` string.  

Next, we need to determine which portion of the number the caret is currently in, the whole number part or the decimal part.  We do this by comparing it's position to the position of the "." in the string.  Note that we care about where the caret was BEFORE the change would have been applied, so we're going to get it's position from `getControlText()`.  

Finally, we set the caret just to either side of the ".", and then set anchor to either the beginning or the end of the string.  This will create a selection in the `TextField`.

Once again, since we're not changing the contents of the `TextField` string, there's no need to run it through the regex to see if it's in the correct format.

Now the `Filter` looks like this, with the new logic inserted and the console output removed:

``` java
public class FixedDecimalFilter implements UnaryOperator<TextFormatter.Change> {

    @Override
    public TextFormatter.Change apply(TextFormatter.Change change) {
        int decimalPos = change.getControlText().indexOf(".");
        if (change.getText().isEmpty() && isEverythingSelected(change)) {
            change.selectRange(0, decimalPos);
            return change;
        }
        if (change.getText().equals(".")) {
            change.setText("");
            change.setRange(0, 0);
            if (change.getControlCaretPosition() <= decimalPos) {
                change.setCaretPosition(decimalPos + 1);
                change.setAnchor(change.getControlText().length());
            } else {
                change.setCaretPosition(decimalPos);
                change.setAnchor(0);
            }
            return change;
        }
        if (change.getControlNewText().matches("-?([0-9]*)?(\\.[0-9]*)?")) {
            return change;
        }
        return null;
    }

    private boolean isEverythingSelected(TextFormatter.Change change) {
        return (change.getSelection().getStart() == 0) && (change.getSelection().getEnd() == change.getControlText().length());
    }
}
```
That's all for this part.  We still need to handle the scrolling of the decimal portion as digits are added or removed, and the "-" sign.  We'll do that in Part 3.
