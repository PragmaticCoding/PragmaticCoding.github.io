---
title:  "Unravelling MVC, MVP and MVVM"
date:   2022-05-30 12:00:00 -0500
categories: javafx
logo: /assets/logos/LittleBrain.jpg
excerpt: Taking a look at the three most common design patterns for building systems with user interfaces.  How are they different?  Which one is best?  Is there anything better?
---

# Introduction

Model-View-Controller (MVC), Model-View-ViewModel (MVVM) and Model-View-Presenter (MVP) are three of the most common design patterns that you are likely to encounter when working with systems that have a User Interface (UI).  It's really important to understand how design patterns work, what they achieve and how it implement at least one of them in the environment that you work in.

I have to say that I struggled with this article for a long time.  I worked for years with a pattern that I had assumed was MVC, and felt that it was somehow incomplete so I added some bits on that made sense to me.  Then I learned that my understanding of "Model" was not the same as the accepted definition.  That "correct" definition of Model makes MVC more complete, but I'm not really happy that it maps well to JavaFX.  

Who am I to argue with the giants of application framework theory?  No one, really.  But I can say that my understanding of how to build clean and simple JavaFX has come over nearly ten years of practical experience, working at it pretty much 9-5 every weekday.

## What's Their Purpose

These are *design patterns*.  They are standard ways of architecting things to achieve common ends.  The advantage to using a design pattern is that they are generally well known and well thought-out, so they are familiar and easy to build and to understand when you encounter them in the real world.

In this particular case, these are different approaches to solve the problem of separating the presentation of data to the user from the actual data and the back-end logic.  In this way, the UI becomes much less dependent on the back-end structure (and visa versa) so that changes to either the UI or the back-end don't require huge changes to other end.

# Understanding the Terminology

A small amount of time spent searching the web will show you that there's many different interpretations of what design patterns really mean.  Even if you go back to some of the original articles where these patterns were first introduced, there's a lot of ambiguity and room for interpretation.  

So let's look at the most confusing concept first:

## The "Model"

There's an idea that data should be bundled up with code that is related to it, so that the result is a responsive object rather than just a bunch of data in fields.   This is a common concept in OOP system design, and you've undoubtedly seen it many, many times.  Even if it's just getters and setters to isolate the internal implementation of the data from it's outward presentation.

That's the concept behind "Model" in all of these design patterns.  It's not just the data, but it's the data handling as well.  This means the business logic, and calls to external API's and a persistence layer.  All of it.

Often it's referred to as the "Domain Data"

## Domain Data

This is where we get into more trouble.  For me "domain data" has always meant data related to the context of the application.  For instance, if we're working with an Sales application, then domain data is things like orders, invoices, customers and transactions.

But quite often when looking at the descriptions of these design patterns, it's clear that the meaning of "domain data" can be very contextual.  It's possible that the GUI construct becomes the "domain", and any data related to it, both screen related data and back-end application data.

This seems to be the most common use of "Domain Data" when talking about these design patterns.

# How Do These Patterns Work?

The first thing to understand is that all of these patterns have the same goal, to create independence between the business logic and the presentation that the user interacts with.  They all have a great deal of similarity, but they have some profound differences in the details.

Let's look at each of these patterns and try to figure out the best understanding of what they mean.  We'll start with MVP since it's probably the least useful with modern, Reactive UI design:

## Model-View-Presenter

### Overview

Model-View-Presenter (MVP) has three components which each have very specific roles:

### The Model

As previously discussed, the Model has all of the data, and all of the business logic contained within it.  

### The View

In MVP the View is supposed to be an entirely passive layout entity.  It contains virtually no decision-making logic of it's own, and it's entire purpose is to put data on the screen, and to receive input from the user.  Once it gets input from the user, its only role is to pass it on to the Presenter, which will decide how to handle it.

### The Presenter

The Presenter works hand-in-glove with the View.  The Presenter has access to the View and the inside components of the View and manipulates them to make the View behave as it should.  For instance, the Presenter is responsible for placing data from the Model into the View components, and visa versa.  The Presenter contains all of the logic for defining how active components like Buttons will behave.

As with any of these frameworks, the line between components can become blurred.  For instance, imagine that you have a ComboBox with three selections, the colours red, green and blue.  The context is such that there will never be any other colours that can be selected, so the colour selection should not be a part of the Model, it's a constraint of the system.  So where does the population of the ComboBox list go?  In the View, or in the Presenter?  You could make a case for either way.


## Model-View-ViewModel

### Overview

At first glance MVVM appears to be the best match for modern, reactive UIs as it specifically talks about "binding" to connect the View to the ViewModel.  

### The ViewModel

The ViewModel contains the data in a structure appropriate to bind it to various components of the View.  It also contains the functionality to prepare the data in that format.  In order to do this, the ViewModel needs to access to the data elements stored in the Model, and will probably call methods in the Model to initiate data storage and retrieval.

### The Model

The Model in this pattern *potentially* leans more towards dealing with application domain data, as the ViewModel handles some amount of the transformation of this data into presentation data.  From that perspective, the business logic is split between the Model and the ViewModel, with the Model handling persistence and external API's.

### The View

The View in MVVM is much more active than in MVP.  

## Model-View-Controller

In the traditional, non-Reactive, implementation of MVC the View is generally seen as only a consumer of the Model's data, and the Controller is responsible for taking any updates from the User, performing business logic upon them and updating the Model.

### The Model

The Model contains *all* of the data in the framework.  This includes the elements of State, plus any domain objects that are required for the business logic.  It also contains all of the logic to interface with external parts of the system such as databases or external API's.

### The View

The View contains all of the elements of the UI.  It has a reference to the Model, so it is able to access the data in the Model that contain the components of the State of the UI.  There is no particular requirement that the State elements of the Model are bound to the View components in this framework.  Obviously, in a Reactive environment they would be.

### The Controller

The Controller handles all of the active parts of the UI, and contains the business logic of the UI.

# The Problem With All These Frameworks

I feel that there's something wrong with all of these frameworks.  They break the "Single Responsibility Principle".

In every one of these frameworks, some component is responsible for way more than one thing.  In MVC and MVP, it's the Model which contains not just the Presentation Data but all of the business logic and the domain objects as well.  In MVVM, it's the ViewModel which contains not just the Presentation Data but all of the code to convert domain objects into Presentation Data that can be bound to the View components.



# A Better Framework for Reactive Systems: MVCI

As I said earlier, I went for years misunderstanding the "correct" structure of MVC.  My understanding was the the "Model" was simply the "State" of framework.  This is also sometimes called the "Presentation Model".  

In this interpretation, the Model is simply a POJO with the fields composed of Observable objects, suitable for binding to the various properties of the View elements.  It's just a place to gather all of the elements of State together in one object, so that it may be passed around as a single component.  

As I also said earlier, I always felt that there was something missing - the piece that connects it to the back-end of the application.  So I needed to add that on, and then build some rules about what goes where and who is responsible for what.  The result is something I call Model-View-Controller-Interactor (MVCI) and I've found it to be perfect for use in a JavaFX Reactive implementation.

Let's look at the components:

## The Model

As I've already mentioned, the Model is really just a POJO.  The fields are, for the most part, Observable objects.  This means that they can be Properties, ObservableLists, Bindings or ObservableValues of some sort.  

Originally, I was pretty stringent about following the JavaFX "Bean" structure for getters and setters, and having a getter for both the Observable itself and it's contents.  I'm not sure about the need for that any more, and currently I lean towards just having a getter for the Observables themselves.  The cost for this is that code in the Interactor, which is the only place that getters and setters for the Observable contents is used, gets a little bigger as it needs to do a get() or set() on the Observables.  

I really don't think that relationships between the data elements in the Model should be established in the Model itself.  These relationships generally reflect business rules of some sort, and that kind of logic belongs in the Interactor at all times.  My guide has always been, "If you are looking for business logic, it should always be in one place."  This means that if you are looking to understand how two data elements are related, you don't have to go hunting through the entire framework to find place where they are linked, it will be in the Interactor.

## The View

The View handles every aspect of the User Interface and nothing else.  Properties of the components of the View are bound to each other and to the Properties in the Model.  In this manner, the Model now becomes an accurate representation of the State of the UI.  

## The Controller

The Controller is the "core" of the framework.  It is responsible for instantiating all of the other components and connecting them together.  The Controller connects the MVCI construct to the rest of the application as it contains the only method accessible by any object outside the MVCI construct.  Typically, it will have a method with a name like getView() which will return the View which can then be placed inside another layout or Scene.

The Controller is responsible for handling all of the "actions" in the UI.  This means that anything more active than simply updating the Model (through Bindings in the View) is processed through the Controller.  The Controller defines how these actions will be handled, and any threading that might be required.  In practice, this means that the Controller will be responding to EventHandlers trigger from the View, and will establish any Listeners on the Model.

## The Interactor

The Interactor contains all of the business logic of the framework.  This is the class where both the Model and the domain objects co-exist:  

Business Constraints and Connections Between Model Elements
: Often there are connections between fields defined in the Model which are related to business rules.  For instance, you might have a BooleanProperty that indicates that some combination of other Properties in the Model are invalid.  This Boolean property would be bound by the Interactor to a Binding that it constructs.  This keeps the business logic inside the Interactor.

Connections to External APIs
: If there are going to be connections to external APIs, then they are going to be in the Interactor.

Connections to Data Persistence
: Once again, if you are going to have a direct connection to a persistence layer, then they will be in the Interactor.

Connections to Shared Application Libraries
: In any system with more than one screen, you are likely to have a fair amount of shared business logic or brokers that connect to APIs or persistence libraries.  These libraries are likely to return and accept application domain objects.

Business Logic for Actions
: This is probably the main function of an Interactor: To provide the methods that bring together State, domain data and business rules to make the framework actually do something.  

## Why is this Better?

In my opinion, it's better because each component does one job, and that job matches with its name:

View
: The View is solely and uniquely responsible for the UX.  Nothing more.

Controller
: The Controller is responsible for "controlling" everything.  It's sets it all up, and then acts as traffic controller for any actions that take place, defining how they will be handled and on what thread.

Model
: The Model is the sole repository of the "State" of the UI.  It's just data and has no logic.

Interactor
: The Interactor is business logic element and, as such, is solely and uniquely responsible for *interacting* with the outside world (other than the UX).

This structure makes it simple to understand.  There's never any doubt about what code goes where.

In a way this is very similar to the MVC with State split out into its own element and used Reactively.  Now the Model from MVC minus State becomes the Interactor, and the View directly interacts with State to keep the View & Model (State) synchronized at all times.  The Controller continues to handle actions from the View, but doesn't contain any business logic and delegates that to the Interactor.

# A Simple MVCI Example

Let's look at a simple example that shows how it all goes together, an application that queries an external API to retrieve weather information for a location.  

When it's running, it looks like this:


## The Controller

Here's the Controller code:

``` java
public class WeatherController {

    private final WeatherInteractor interactor;
    private final WeatherViewBuilder viewBuilder;

    public WeatherController() {
        WeatherModel viewModel = new WeatherModel();
        interactor = new WeatherInteractor(viewModel);
        viewBuilder = new WeatherViewBuilder(viewModel, this::fetchWeather);
    }

    private void fetchWeather(Runnable postFetchGuiStuff) {
        Task<Void> fetchTask = new Task<>() {
            @Override
            protected Void call() {
                interactor.checkWeather();
                return null;
            }
        };
        fetchTask.setOnSucceeded(evt -> {
            interactor.updateWeatherModel();
            postFetchGuiStuff.run();
        });
        Thread fetchThread = new Thread(fetchTask);
        fetchThread.start();
    }

    public Region getView() {
        return viewBuilder.build();
    }
}
```
We can see in the constructor that the Interactor and the ViewBuilder are instantiated with the Model as a constructor dependancy satisfied by the newly instantiated WeatherModel.  In addition, the ViewBuilder is supplied a Consumer to handle the "Fetch Weather" action.  

The "Fetch Weather" action requires connection to an external website and is therefore blocking in nature.  That can't happen on the FXAT, so the Controller creates a Task to handle the threading required.  

This controller has exactly one public method other than its constructor, getView().  The method invokes the ViewBuilder build() method to return a Region.

## Connecting it to the Application

We can look at the Application class to see how the Controller supplies the connection from the MVCI framework to the rest of the JavaFX application.  There's nothing complex here, as this application just has the one screen, so we can instantiate the Controller and call its getView() method to populate the Scene:

``` java
public class WeatherMain extends Application {

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        primaryStage.setScene(new Scene(new WeatherController().getView()));
        primaryStage.show();
    }
}
```

## The Model

The Model is, as expected, just a POJO with a bunch of JavaFX property "beans" :

``` java
public class WeatherModel {

    private final StringProperty temperature = new SimpleStringProperty("");
    private final StringProperty conditions = new SimpleStringProperty("");
    private final StringProperty city = new SimpleStringProperty("Nice");
    private final ObjectProperty<Image> icon = new SimpleObjectProperty<>();
    private final ObservableList<String> cities = FXCollections.observableArrayList();
    private final StringProperty units = new SimpleStringProperty("metric");

    public String getTemperature() {
        return temperature.get();
    }

    public StringProperty temperatureProperty() {
        return temperature;
    }

    public void setTemperature(String temperature) {
        this.temperature.set(temperature);
    }
    .
    .
    .
}
```
I've trimmed out most the methods as it just repeats for each Property.

## The ViewBuilder

Here's the ViewBuilder.  The build() method returns a Region which is actually a BorderPane, under-the-hood.  It only has the centre and bottom zones populated.  The centre zone is comprised of a weather image and a GridPane with details.  There are a VBox and an HBox to give some structure to the layout.

The bottom has a city selection ComboBox and a Button in an HBox.  You can see how the onAction() method of the Button invokes the Consumer passed from Controller to fetch the weather, while disabling and enabling the Button at the right times.

``` java
public class WeatherViewBuilder implements Builder<Region> {

    private final WeatherModel viewModel;
    private final Consumer<Runnable> weatherFetcher;

    public WeatherViewBuilder(WeatherModel viewModel, Consumer<Runnable> weatherFetcher) {
        this.viewModel = viewModel;
        this.weatherFetcher = weatherFetcher;
    }

    @Override
    public Region build() {
        BorderPane results = new BorderPane();
        results.setCenter(setUpCentre());
        results.setBottom(setUpBottom(weatherFetcher));
        results.getStylesheets().add(Objects.requireNonNull(getClass().getResource("/css/default.css")).toExternalForm());
        results.getStyleClass().add("weather-box");
        return results;
    }

    private Node setUpCentre() {
        ImageView iconImageView = new ImageView();
        iconImageView.imageProperty().bind(viewModel.iconProperty());
        iconImageView.setFitWidth(200);
        TwoColumnGridPane gridPane = new TwoColumnGridPane();
        gridPane.addTextRow("Temperature:", viewModel.temperatureProperty());
        gridPane.addTextRow("Conditions:", viewModel.conditionsProperty());
        gridPane.setMinHeight(160);
        gridPane.setMinWidth(200);
        VBox vBox = new VBox(5, TextWidgets.boundStyledText(viewModel.cityProperty(), "heading-text"), gridPane);
        HBox hBox = new HBox(10, iconImageView, vBox);

        vBox.getStyleClass().add("info-box");
        vBox.setPadding(new Insets(6));
        return hBox;
    }

    private Node setUpBottom(Consumer<Runnable> fetchWeather) {
        Button button = new Button("Get Weather");
        button.setOnAction(evt -> {
            button.setDisable(true);
            fetchWeather.accept(() -> {
                button.setDisable(false);
            });
        });
        ComboBox<String> cityCB = new ComboBox<>();
        cityCB.getItems().setAll(viewModel.getCities());
        viewModel.cityProperty().bind(cityCB.valueProperty());
        ToggleButton unitsToggle = new ToggleButton();

        return new HBox(10, cityCB, button);
    }
}
```

## The Interactor

Now, let's look at the Interactor:

``` java
public class WeatherInteractor {

    private final WeatherModel viewModel;
    private final WeatherFetcher weatherfetcher = new WeatherFetcher();
    private WeatherData weatherData;

    public WeatherInteractor(WeatherModel viewModel) {
        this.viewModel = viewModel;
        viewModel.setCities(Arrays.asList("London", "Paris", "Nice", "Hong Kong", "New York", "Las Vegas"));
    }

    public void checkWeather() {
        weatherData = weatherfetcher.checkWeather(viewModel.getCity());
    }

    public void updateWeatherModel() {
        viewModel.setTemperature(weatherData.getTemperature());
        viewModel.setConditions(weatherData.getConditions());
        viewModel.setIcon(weatherData.getWeatherImage());
    }
}
```
The constructor holds any initialization of the Model that's required.  In this case, it's the list of cities that weather can be checked for.  It's clear that this list is absolutely "business logic" for this application, and the responsibility for initializing it belongs here, in the Interactor.

The two public methods are involved in the "Fetch Weather" action, and are called from the Controller via its Task.  The first method, checkWeather() is intended to be called from a background thread.  The second is intended to be called on the FXAT, and therefore, can update the Model.  

We're using an internal library, WeatherFetcher, as a "broker" or a "service" to query the web to get the weather.  This broker returns WeatherData, which is a "domain" object.  If this was a bigger application with lots of different functions and screens, then there would be a whole application layer with brokers and utilities to perform functions and apply business logic shared across the entire application.  But here we just have a single broker, which we're not going to look into at all since it's not relevant to the MVCI framework.

Lastly, you can see how the domain object is stored in the Interactor as a field.  This way it can be populated on the background thread and then accessed on the FXAT.  Obviously, with two threads accessing the same data, there are concurrency issues that need to be addressed.  In this particular example, there is no way that the we can have problems because the structure around the action prevents multiple concurrent Button clicks.

# The Value in the Framework

Now we have a concrete example, let's look at how this framework adds value to the application.

It really just does one important thing: It reduces coupling.  

## How much coupling do we have?

The View has no public methods other than those inherited from Region.
: Public methods are a primary indication of coupling.  Here we have absolutely zero public methods specific to the View, so there's no coupling at all in this manner.   This means that there are no dependencies on the View!

The View's only dependencies are declared in its constructor.
: The View is dependant on the Model and the Consumer passed to it from the Controller. The View has no knowledge of the existence of either the Controller or Interactor.

The Interactor's only dependency is on the Model.
: The Interactor has no knowledge of the View or the Controller.  It has dependencies on the structure of the Model, but that's it.  

The Controller has dependencies on the Interactor.
: Since the Interactor has two public methods that the Controller needs to call, there are dependencies there that the Interactor needs to satisfy.  

The Controller might have dependencies on the Model.
: While this example doesn't have any Controller dependencies on the Model, it's possible that it could.  

The View has no coupling on the data in the Model, Just its structure.
: In this example, the List of cities is populated by the Interactor, and then loaded into the ComboBox in the View.  The View doesn't have any logic dependant on the contents of that list.  If we wanted to create such a dependency, we'd have to replace the List with something like an Enum, so that the possible values are now part of the *structure* of the data, not it's contents.

Obviously, in any non-trivial system there has to be at least some level of coupling if the system is going to be functional.  In MVCI the Model is the key component of coupling, as all three of the other components of the framework are dependent on it.  

## Swapping Out Components

I find that a mental exercise that's helpful to understand the impact of coupling is to imagine replacing the View or the Interactor with something fantastically different, just to see what the impact would be on the system.

What if we replaced the View from a simple screen to a "Weather Experiencing Chamber?"  Imagine you walk into a room and there are a bunch of buttons on the wall, one for each city, and when you hit one it then changes the temperature in the room and blows air, or showers water on you, turns on a fog machine.  Things like that.  

What would you need to change in the rest of the system to make it work?  Really, nothing.  The UX is completely divorced, decoupled if you like, from the mechanics of fetching the data.  You still need a list of cites, and you still need to update the Model with the selected city and trigger the action.  And you still need to respond to the changes in the Model that hold the weather information.

Now, let's look at it the other way around.  What if we now have a "Weather Setting" system, instead of a "Weather Fetching" system.  Pick a city, set the temperature and click a button.  Now our system of weather controlling satellites changes the temperature in Paris.  In this case, we'll need to change the GUI so that the temperature is an input, not an output.  The Interactor can be changed with a new Interactor that has an interface to our satellite network that controls the weather.  The Model does not need to change, nor does the Controller.  The change to the GUI is minimal, and isn't particularly tied to the functionality contained in the Interactor.  
