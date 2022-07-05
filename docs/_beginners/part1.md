---
title: Part 1 - A Quick Start
short_title: Part 1
permalink: /beginners/part1
excerpt: The very first step.  What else?  "Hello World" in JavaFX.  

hello: /assets/beginners/hello.png
gradle: /assets/beginners/build.gradle
---

# What You'll Learn

1. How to get a JavaFX project up and running fast.
1. The `Application` class.
1. The `Stage` class.
1. The `Scene` class.
1. Your first `Nodes`: `Label` and `Region`
1. A little bit about organizing your code.

# The Fastest Start Possible

The quickest way to get started is to download this [build.gradle]({{page.gradle}}) file and put it in a directory all by itself.

Now, open it with Intellij Idea.  I'm sure you can do the same thing with Eclipse, too.

This should start up a new project.  Let it build the Gradle environment and initialize the project.  The select the "build->build" option from the Gradle menu.

Now go to the top level of the project and select, "New Directory".  The dialogue box should give you option for the Gradle defined source directories.  Highlight them all and hit `<Enter>`.

Next, if you want to change the group information in `build.gradle` from "ca.pragmaticcoding", you should do it now.  Then go to the `\src\java` directory and select "New -> Package".  You should create something like "ca.pragmaticcoding.beginners.part1".

You'll need a `module-info.java` file in the root of the `java` directory.  It should look like this:

``` java
module ca.pragmaticcoding.beginners {
    requires javafx.controls;
    requires javafx.graphics;
    requires javafx.base;

    exports ca.pragmaticcoding.beginners.part1;
}
```

Finally, select that package and then use the menu "New -> Java Class", and call it "Main".  Copy the following into it:

``` java
import javafx.application.Application;
import javafx.stage.Stage;

public class Main extends Application {

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {

    }
}
```


And there you go!  Now we have the skeleton of your first JavaFX application.

# Application, Stage and Scene

Let's take a quick look at this little bit of code, because there's more going on there than it seems.

## Application

`Application` is an abstract class that has all of the guts that gets the JavaFX engine up and running for you.  This is all started up when the `launch()` method is called.  Eventually, `launch()` will call `start()` which is an abstract method.  We're going to be putting all of our customized code for the application in here.

## Stage and Scene

The most mystifying thing when you start with JavaFX is all this `Stage` and `Scene` stuff.  Simply put, the `Stage` is the element that corresponds to the window on your screen.  If you want to have multiple windows, you'll generally need to have multiple `Stages`.

`Scene` is the component that holds the contents of your window.  Why is `Scene` separate from `Stage`?  I don't honestly know, and I don't know what advantage is gained by having the functionality split between two classes.  

Ordinarily, you'll create a Scene and then put a JavaFX `Region` subclass `Node` inside it.  Then you put the `Scene` inside the `Stage`, and call the `Stage's`, `show()` method.

So let's do that:

``` java
import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.control.Label;
import javafx.scene.layout.Region;
import javafx.stage.Stage;

public class Main extends Application {

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        Scene scene = new Scene(createContents());
        primaryStage.setScene(scene);
        primaryStage.show();
    }

    private Region createContents() {
        return new Label("Hello World");
    }
}
```

# Running the Application

You can run it now.  I'd suggest using the "Application -> Run" from the Gradle menu.  The output should look like this:

![Hello]({{page.hello}})

Not very exciting really, but it **is** something that runs and opens a window.

Two little pieces of explanation first, and then we'll conclude Part 1.

You'll notice that I've split `createContents()` out from the rest of the code in `start()`, even though it's really simple.  Personally, I think `start()` should be all about `Scene` and `Stage` and getting the window on the screen.  The details about the contents of the `Stage` really don't belong in `start()`.  Keeping methods small and purposeful is just as important with JavaFX as it is with any other programming.

You should also notice that `createContents()` returns `Label` as a `Region`.  Strangely enough, all of the `Control` classes in JavaFX, including `Label` extend from `Region`, even though they don't seem at first to be `Region` kind of objects.  In the specific case of `Label`, there's actually more going on to its structure than you think, and it actually does make sense to think of it as a `Region`.  We'll look at that later.
