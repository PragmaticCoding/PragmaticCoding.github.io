---
title:  "The Real Difference Between Swing and JavaFX"
date:   2022-06-19 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
ScreenSnap: /assets/posts/StarterFX.png
GitHubDialog: /assets/posts/StarterFX-GitHub.png
Diagram: /assets/posts/MVCI.png
excerpt: You're starting a desktop GUI project and you want to know which is the best toolkit to use, Swing or JavaFX.  How are they different? What's the advantages of one over the other?  Let's take a look.
---

# Introduction


# A Look at Swing Properties

The primary support for Observable Properties in Swing is through PropertyChangeSupport.  This allows you to add and remove listeners to class, and to fire change events when a field is modified.  

Here's some sample code, straight from the JavaDocs for Swing:

``` java
public class FaceBean {
    private int mMouthWidth = 90;
    private PropertyChangeSupport mPcs = new PropertyChangeSupport(this);

    public int getMouthWidth() {
        return mMouthWidth;
    }

    public void setMouthWidth(int mw) {
        int oldMouthWidth = mMouthWidth;
        mMouthWidth = mw;
        mPcs.firePropertyChange("mouthWidth", oldMouthWidth, mw);
    }

    public void
    addPropertyChangeListener(PropertyChangeListener listener) {
        mPcs.addPropertyChangeListener(listener);
    }

    public void
    removePropertyChangeListener(PropertyChangeListener listener) {
        mPcs.removePropertyChangeListener(listener);
    }
}
```
There are some important things to note about this:

Everything is Manual
: There's nothing intrinsic about the observability of the anything here.  You have to write the code to fire property change event, and you have to add the methods to allow other classes to register listeners with this class.

Property Changes are Events
: This is not a data-centric structure.  When the value changes, the code fires an event, and it's up to the event to figure out how to deal with what has happened and update any data values.

The Observable Nature is Established at the Wrapping Class Level
: Notice that PropertyChangeSupport is associated with FaceBean, not mMouthWidth.  Notice that setMouthWidth passes an extra parameter that tells the listener that it was "mouthwidth" that changed.  If there were more fields that were added to FaceBean they would all share the same PropertyChangeListeners.

The implications of the last point are huge.  It means that the fields have no observable nature of their own, and cannot be detached from the wrapping class and remain observable.  Any class that wishes to treat any fields in the wrapping class as observable needs to take the entire wrapping class.


Now let's look at how you are supposed to create a PropertyChangeListener.  Once again, from the JavaDocs for Swing:

``` java
//...where initialization occurs:
double amount;
JFormattedTextField amountField;
...
amountField.addPropertyChangeListener("value", new FormattedTextFieldListener());
...
class FormattedTextFieldListener implements PropertyChangeListener {
    public void propertyChanged(PropertyChangeEvent e) {
        Object source = e.getSource();
        if (source == amountField) {
            amount = ((Number)amountField.getValue()).doubleValue();
            ...
        }
        ...//re-compute payment and update field...
    }
}
```

Like all of the Swing widgets, JFormattedTextField implements PropertyChangeSupport just like FaceBean in the first code sample does.  In this case, the JFormattedTextField has a field called value, and the listener is added to catch events that are fired from that field.

A couple of things first:  This example code seems to be old and predates the introduction of lambdas to Java. So, while PropertyChangeListener is a functional interface, it's been build out old-style.  Probably for the same reason, FormattedTextFieldListener is written rather generically, probably with the intent that it could then be expanded to handle events from a number of other TextFields in this GUI.  The result is something that's quite a bit more complicated it probably needs to be.  

If I was writing such a thing today, it would probably look like this:

``` java
//...where initialization occurs:
double amount;
JFormattedTextField amountField;
...
amountField.addPropertyChangeListener("value",evt -> amount = ((Number)amountField.getValue()).doubleValue());
```

That's a lot simpler, and has no cost even if you add a lot more JFormattedTextFields to the GUI.

In the simpler implementation, you can see that the PropertyChangeListener is working essentially as a trigger to cause the amount variable to be updated from amountField whenever the value field in amountField changes.

# Creating a JavaFX Property

What if we created a utility class to be a wrapper around a particular, single value that we want to make Observable.  This would look a lot like FaceBean, above, but would more generic in use:

``` java
public class SwingStringProperty {
    private String value = "";
    private PropertyChangeSupport mPcs = new PropertyChangeSupport(this);

    public String get() {
        return value;
    }

    public void set(String newValue) {
        String oldValue = value;
        value = newValue;
        mPcs.firePropertyChange("value", oldValue, newValue);
    }

    public void
    addPropertyChangeListener(PropertyChangeListener listener) {
        mPcs.addPropertyChangeListener(listener);
    }

    public void
    removePropertyChangeListener(PropertyChangeListener listener) {
        mPcs.removePropertyChangeListener(listener);
    }
}
```
