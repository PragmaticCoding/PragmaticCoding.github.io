---
title: Part 5 - Application Framework
short_title: Part 5
permalink: /beginners/part5
screenshot_1: /assets/beginners/part4_1.png
excerpt: Building an application that "does something" means adopting a framework that works well with JavaFX.  Here we build the skeleton of the Model-View-Controller-Interactor framework that we're going to use for our CRUD application.
---

# What You'll Learn

1. The importance of having an application structure.
1. How to build a bare-bones structure that runs.


# Creating a Framework

In order to organize our application so that the GUI will be separated from the business logic, and to facilitate the creation of a Reactive application, we'll need to adopt a framework.  There are lots to choose from, but I'm going to use one that I've developed myself that I find works very well.

This framework is called Model-View-Controller-Interactor (MVC-I).  Rather than spend a lot of time talking about how it works, we'll just start by cooking it up as simple as possible and it should be clear how things go as we build it out in our application.

But, to start with, we'll need the skeleton to build everything on.  So here we go.

# The Application Itself

Since this is JavaFX, we still need to have a class that extends `Application` and one that implements `main()`.  So, we'll call it `Main` and add what we need to start using our framework.

``` java
public class Main extends Application {
    @Override
    public void start(Stage primaryStage) throws Exception {
        primaryStage.setScene(new Scene(new CustomerController().getView()));
        primaryStage.show();
    }
}
```
This is about the simplest `Application` implementation that you can have.  We instantiate a Controller and then call its `getView()` method to get the "root" for our `Scene`, which we put into the `Stage`.  Then we call `Stage.show()`.

# The Controller

``` java
public class CustomerController {

    private Builder<Region> viewBuilder;
    private CustomerInteractor interactor;

    public CustomerController() {
        CustomerModel model = new CustomerModel();
        viewBuilder = new CustomerViewBuilder(model);
        interactor = new CustomerInteractor(model);
    }

    public Region getView() {
        return viewBuilder.build();
    }
}
```
From this you can see that the Controller is responsible for instantiating all of the other parts of the framework.  The Model is passed to the constructors of the ViewBuilder and the Interactor.  This way all of the elements have a reference to the Model.  You can see that `getView()` just delegates to the `build()` method of the ViewBuilder.


# The View Builder

``` java
public class CustomerViewBuilder implements Builder<Region> {

    private final CustomerModel model;
    public CustomerViewBuilder(CustomerModel model) {
        this.model = model;
    }

    @Override
    public Region build() {
        return new VBox();
    }
}
```
There's a key point hidden away here.  Since we are only going to use our View as standard descendent of `Region`, we don't need, nor do we want, a custom class extending some container `Node`, like `VBox`, `HBox` or `BorderPane`.  It will be customized, but we're not going to add any new features to it (which really means adding public methods), so just returning `Region` is all we want.  

To do this, we use a `Builder`.  So here you have a `Builder`.  Right now, it just builds an empty `VBox`.  

# The Interactor

``` java
public class CustomerInteractor {
    private CustomerModel model;

    public CustomerInteractor(CustomerModel model) {
        this.model = model;
    }
}
```
Just the bare-bones here.  A class with the appropriate constructor, but nothing else.

# The Model

``` java
public class CustomerModel {
}
```
Even more bare-bones.  We need this class, but it hasn't come into play yet.  

# This Will Run

Yep.  It's complete, and you get an empty window on the screen.  Woo-hoo!

Okay, so there's not much here, but...

You should be able to see how all three of the other components share a reference to the `Model` so it's going to be a way for them to communicate with each other without actually knowing about each other.  That's a crucial concept.  

You should also be able to see that the Controller knows about both the ViewBuilder and the Interactor, yet neither of those classes knows about the Controller.
