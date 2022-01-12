---
title:  "Implementing MVC in JavaFX"
date:   2021-05-11 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/MVC_In_JavaFX
excerpt: Model-View-Controller is generally accepted as a good way to structure an application with a user interface.  Since JavaFX implements Reactive programming there's a natural way to incorporate MVC into a JavaFX application.  However, it doesn't seem to be widely documented and there's little evidence on the Web that many people have figured out how to do it properly.
---

Model-View-Controller is generally accepted as a good way to structure an application with a user interface.  While JavaFX contains all of the classes needed to implement MVC in a natural and seamless way, it doesn't seem to be widely documented and there's little evidence on the Web that many people have figured out how to do it properly.  The fact that JavaFX supports Reactive programming makes a big difference to how you should implement MVC.

## Model-View-Controller-Interactor

One of the biggest complications with MVC in JavaFX is the need to handle background threads to perform time intensive work which cannot take place on the Application Thread (FXAT).  Coupled with that is a firm rule that GUI elements, including any `Observable` type variables and fields must only be updated by code running on the FXAT.  

To make this easier to handle, I prefer a slight modification to MVC which splits the controller into two parts, adding an "Interactor" which handles the business/application logic for the screen.  You don't need to do this split if you don't want to, but it does make the code more organized.

Let's look at what each part does:

### The Model

The model is the storage space for all of the data shared throughout the the MVCI structure.  It  is instantiated by the Controller, which retains a reference to it, and passes that reference on to the View and the Interactor.  It is the only source of direct communication between the View and the Interactor.

In the world of Reactive Programming, the Model is the home for the elements generally referred to as "State".

In JavaFX, the building blocks of data storage are "observable" entities such as Properties and ObservableLists.  The Model should be a POJO with the fields all instances of the various observable classes in JavaFX.  Each property type field should have a getter and a setter which delegate to the `get()` and `set()` methods of the property, and a `{Field Name}Property()` method which returns the property itself.  Essentially, this is the JavaFX version of a Bean.  Normally, an ObservableList type field would have a getter, and then a setter that delegates to the `setAll()` method of the list.  All of the fields should be final.

### The Controller

The Controller is the central element of the structure, and the connection to the rest of the application's GUI.  It's instantiated first, and is responsible for instantiating the other components and handling the communication between them.  It's also responsible for handling any threading, which is important in JavaFX as all non-GUI activity needs to happen in a background thread.

### The View

The View is the actual graphical content of the screen.  Typically, the View would be a class which extends one of the Region type Nodes in JavaFX; like a BorderPane, VBox or StackPane.  Something which can be inserted into a Scene.  The View is restricted to having only code which is directly related to displaying information on the screen and accepting user input.

The View normally interacts with the Model in one way only.  It binds the properties in the model to the properties in the JavaFX nodes on the screen.  User editable fields are bound bidirectionally, and display only fields are bound from the Model to the value properties of display nodes.  In this way, the Model always reflects the current state of the information displayed on the screen, and there is no reason to write code that scrapes the data out of the screen nodes to submit it to the Controller when any action button is clicked.


### The Interactor

The Interactor is the application - or business - logic of the MVCI system.  It's the piece which "interacts" with the non-GUI elements of the application.  It can invoke the persistence layer or communication classes to talk to other systems, and it can deal with both domain objects and the Model.  It's responsible for establishing interrelationships between elements of the model based on business rules.

Generally speaking, the Interactor is the only class which should be allowed to use the getters and setters in the Model.  The Controller and the View should only access the fields by the `...Property()` methods - and then usually just to bind them or add listeners.

### How Does the Interactor Help?

While it's really just an organizational thing, it's critical to providing structure to the application by isolating the application logic from screen control logic.  Technically, everything that the Interactor does could be handled by the Controller, but at the cost of making the Controller much more difficult to understand.

The Interactor is where the domain objects, process data and screen data all exist in the same place.  Typically, an Interactor will make calls to a persistence layer or external API and then it will need to store the results from those calls.  It might have data structures to handle processing and analysis of the domain objects specific to the functionality of the MVCI component, all of which need to be updated on a background thread.  In addition, it will probably need to *read* the contents of the Model in order to make those API calls and process the resulting data on that background thread.


### Keeping Things in the Right Place

One of the keys to writing clean applications, second only to naming things clearly, is putting code in the right place.  The toughest part of MVCI is keeping business logic out of the View, as it is extremely easy for it to sneak in.  One of the best ways to avoid this is to follow a strict rule of not creating relationships between screen nodes unless the relationship is completely about display.  

For instance, if there is a checkbox that makes a region of the screen visible or invisible, then it's OK to bind the visible property of the region to the value property of the checkbox.  But if the visibility of that region is based on something in the data, perhaps a threshold value for a particular field, then that linkage can't be established by the View itself, and needs to come from the Model.

How do you put that in the Model?  

Create a boolean property in the Model called something like, "yadaYadaThresholdMet", and bind the visibility of the region to that field.  The logic to actually decide if the threshold has been met belongs in the Interactor, and there should be some code, generally invoked from the Interactor's constructor, that binds that boolean property to other fields in the Model properly.

Philosophically, it's also important not to call that boolean property something like, "showXyzRegion", as this has no meaning when someone is looking at the Interactor.  Since it's actually expressing some sort of application rule, it should be named in a manner that clearly reflects that rule.

While the View has no code which reflects business logic, no other part of the system can ever assume anything about the contents of the View.  Any particular field in the Model may or may not be present in the GUI, and manner in which it's displayed cannot be known.  There may or may not be a "Save" button - maybe the save action is triggered by changing the value in a ComboBox?

## A Simple Example

Here's a simple example to illustrate how this all goes together.  This is an MVCI set-up with screen that has a TextField and a button, and the button triggers an action that will modify the value in the TextField in some way - in this case it assumes it is a number and attempts to add 5 to it.

{% highlight java %}
public class MvciSample extends Application {

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        Scene scene = new Scene(new MvciController().getView(), 500, 200);
        primaryStage.setScene(scene);
        primaryStage.show();
    }
}
{% endhighlight %}

You can see here that the application instantiates the Controller, then fetches the View from it and installs it into the Scene.  Let's look at how the Controller works:

{% highlight java %}
public class MvciController {

    private final Region view;

    public MvciController() {
        MvciModel viewModel = new MvciModel();
        MvciInteractor interactor = new MvciInteractor(viewModel);
        view = new MvciView(viewModel, interactor::addFive);
    }

    public Region getView() {
        return view;
    }
}
{% endhighlight %}
The Controller instantiates the Model, then passes it to the constructor of the Interactor and then the constructor of the View.  Additionally, the Controller passes a Runnable to the View to handle the action when View triggers it.

Now let's look at the Model:

{% highlight java %}
class MvciModel {

    private final StringProperty number = new SimpleStringProperty("0");
    private final BooleanProperty moreAllowed = new SimpleBooleanProperty(true);


    String getNumber() {
        return number.get();
    }

    StringProperty numberProperty() {
        return number;
    }

    void setNumber(String number) {
        this.number.set(number);
    }

    boolean isMoreAllowed() {
        return moreAllowed.get();
    }

    BooleanProperty moreAllowedProperty() {
        return moreAllowed;
    }

    void setMoreAllowed(boolean moreAllowed) {
        this.moreAllowed.set(moreAllowed);
    }
}
{% endhighlight %}
There are just two fields, one is a StringProperty that holds a number, and the other is a BooleanProperty that will control whether the "Add 5" action is allowed at any time.  Both fields are explicitly declared final, but they're also effectively final since the "setter" for each actually delegates to the set method of the properties.  

Now to look at the View:

{% highlight java %}
class MvciView extends VBox {

    MvciView(MvciModel model, Runnable actionHandler) {
        TextField numberTF = new TextField();
        numberTF.textProperty().bindBidirectional(model.numberProperty());
        HBox dataBox = new HBox(4, new Label("Enter a number: "), numberTF);
        Button button = new Button("Add 5");
        button.setOnAction(evt -> actionHandler.run());
        button.disableProperty().bind(model.moreAllowedProperty().not());
        getChildren().addAll(dataBox, button);
    }
}
{% endhighlight %}
The View is one of the simplest JavaFX container nodes, a VBox.  There are two elements in the VBox; an HBox containing a label and a TextField for data entry, and a button that performs an action.  The "text" property of the TextField is bound to the `number` field in the Model, and the "disable" property of the button is bound to the `moreAllowed` field in the Model.  The button has been set up so that it's `onAction()` event handler will invoke the `actionHandler` Runnable.  

The only constraints on the structure of the View are defined by the dependencies in the constructor.  It must be able to function with the Model it's given, and potentially invoke the action supplied through the Runnable.  The TextField doesn't have to be a TextField, potentially it could be a Spinner (although it would have to handle conversions to String in the binding) or some other input method.  Presumably, if it was a ComboBox, the Model would have to be able to supply the possible value choices somehow.

Now, the Interactor:

{% highlight java %}
class MvciInteractor {

    MvciModel viewModel;

    MvciInteractor(MvciModel viewModel) {
        this.viewModel = viewModel;
        viewModel.moreAllowedProperty()
            .bind(Bindings.createBooleanBinding(() -> checkIfMoreAllowed(),
                 viewModel.numberProperty()));
    }

    void addFive() {
        try {
            viewModel.setNumber(Integer.toString(Integer.parseInt(viewModel.getNumber()) + 5));
        } catch (NumberFormatException e) {
            viewModel.setNumber("5");
        }
    }

    private boolean checkIfMoreAllowed() {
        try {
            int numberValue = Integer.parseInt(viewModel.getNumber());
            return (numberValue < 21);
        } catch (Exception e) {
            return true;
        }
    }
}
{% endhighlight %}
This interactor does two important things.  First, it provides the logic to perform the "Add 5" action by converting the string into an integer and adding 5.  If the integer conversion fails, then it resets the string to "5".

Secondly, it binds the Model field "moreAllowed" to the "number" field through a supplier that invokes the `checkIfMoreAllowed()` method.  This method always returns true if the string in the "number" field is not a number, so that the `addFive()` method can reset it when invoked.

Note that this Interactor has no knowledge of the structure of the View baked into it.  It deals with the Model alone, and supplies a single callable method with no constraints or dependencies about when or how it will be invoked.  Just like the View, it could be replaced with a different Interactor so long as that new Interactor handles the same model and supplies an `addFive()` method.  For instance, a new Interactor could treat the "number" string like a string and simply concatenate a "5" onto it, setting "moreAllowed" to false if the string had 10 or more characters in it.


## Packaging and Visibility

The best packaging is probably to place all of the MVCI components into the same package so that visibility of the inner parts of the components can be controlled properly.  Any other custom view elements, such as extensions of TableView, can be added to the same package.

Note that the only methods with "public" visibility are the constructor and the `getView()` method of the controller.  All of the other classes are "package-private", and have no exposure from classes outside the package.  Although this example is too simple to see it, it should also be noted that the View has no methods which are not "private" except its constructor.  In other words, nothing of the inner structure of the View is exposed to any other classes.  

In order to ensure that the "set" and "get" methods of the Model are only callable from the Interactor, it's tempting to put the Model and the Interactor in a separate sub-package and leave those methods as "package-private".  But this would mean that the constructors and all of the other externally callable methods in those two classes would need to be public, which would expose them outside of the MVCI structure.  So it's probably best to leave everything in one package and use programmer discipline to ensure that methods are only called in the appropriate manner.  This is made a little bit easier by the fact that the only the Controller has references to all three of the other components, while the View and the Interactor only have a reference to the Model.  The Model, of course, has no references to any other components of the MVCI.

## Enabling "Save" Only When Data Has Been Changed

One situation which is fairly common and appears to be difficult with this approach is when you have a "Save" button that you only want enabled when the user has changed something.  When using the value properties of the screen nodes to store the user data separate from the data model, in seems easier to be able to compare the original values (in the model) to the values on screen.  However, Thomas Nields has written an excellent library called [DirtyFX](https://github.com/thomasnield/DirtyFX) which handles this wonderfully.

Normally, most models have ObjectProperty's which are instantiated using the SimpleObjectProperty implementation.  Thomas has created a set of implementations which also allow the initial state of the property - a baseline - to be set and then it maintains an internal boolean property which indicates whether or not the current value of the property is different from the baseline - if it's "dirty".

Implementing this in the example model is straight-forward:

{% highlight java %}
public class MvciModel {

    private final DirtyStringProperty number = new DirtyStringProperty("0");
    private final BooleanProperty moreAllowed = new SimpleBooleanProperty(true);


    public String getNumber() {
        return number.get();
    }

    public StringProperty numberProperty() {
        return number;
    }

    public void setNumber(String number) {
        this.number.set(number);
        this.number.rebaseline();
    }

    public BooleanProperty moreAllowedProperty() {
        return moreAllowed;
    }

    public ObservableValue<Boolean> dataChanged() {
        return number.isDirtyProperty();
    }
}
{% endhighlight %}

I've removed the setter for the `moreAllowed` field, since the property is bound in the constructor of the Interactor, and a bound property cannot be set.  I've also removed the getter, since the Interactor doesn't use it.  The `dataChanged()` method is only there to expose the `isDirty` property of "number" to the interactor, while keeping the return value of `numberProperty()` to be StringProperty - in other words, not exposing the DirtyStringProperty implementation of the field to the rest of the system.

Note that the setter for the "number" field now calls `rebaseline()`, which resets the baseline value to whatever the current value of the property is, after calling the `set()` method of the property.  This works nicely with the idea that the setter is only called from the Interactor, and the View interacts with the property only through a binding, and therefore won't invoke the `rebaseline()` method when the value in the TextField is changed.  

The constructor of the Interactor now looks like this:

{% highlight java %}
public MvciInteractor(MvciModel viewModel) {
        this.viewModel = viewModel;
        viewModel.moreAllowedProperty()
                 .bind(Bindings.createBooleanBinding(() -> checkIfMoreAllowed() && viewModel.dataChanged().getValue(),
                                                     viewModel.numberProperty(),
                                                     viewModel.dataChanged()));
    }
{% endhighlight %}
Now the "moreAllowed" property will only be true if the number in the field is less than 21 or not actually a number, and has been changed since the the last time the `addFive()` method was invoked.  DirtyFX also has a composite dirty property, which allows you to register other dirty properties with it, and will be true if any of the component properties are dirty.

## In Summary

This example is simple because it lacks any deeper application behind it, and therefore no need to perform background processing to handle communication with a database or external API.  If it did, then the simple Runnable to invoke the `addFive()` method would need to be changed to something more complex to allow the GUI to be modified prior to invoking the background job, perhaps showing some kind of progress indicator and disabling the button, and then to return the GUI back to it's original state when the job was completed.  The Controller would need to be enhanced to create a Task to run the background job, and then to invoke a second Interactor method to update the Model on the FXAT after the background job had been completed.  
