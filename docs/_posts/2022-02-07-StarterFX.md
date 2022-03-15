---
title:  "StarterFX: A Gradle Based JavaFX Ignition Project"
date:   2022-03-30 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/starterfx
ScreenSnap: /assets/posts/StarterFX.png
GitHubDialog: /assets/posts/StarterFX-GitHub.png
Diagram: /assets/posts/MVCI.png
excerpt: How do you start writing a JavaFX application?  Here's a self-contained project guaranteed to run out-of-the-box, with enough structure to get you started.
---

# How Do You Start Writing a JavaFX Application?

It used to be a little bit easier to get started with JavaFX.  Then the JavaFX SDK was split off from the main Java JDK, and Java 9 introduced modules.  Now it's just a little bit more complicated.

There's nothing more frustrating than getting stuck on the very first step.

So here's a project on GitHub that you can download and open up with any IDE that supports Gradle, and it will run as soon as Gradle has finished it's setup and you click on "Application->Run" in the Gradle menu.

# It's Tiny, But it's Powerful

This application has just two classes and a build file, but it saves a lot of frustration.

## What's in the Box?

An application that looks like this when it runs:

![Screen Snap]({{page.ScreenSnap}})


### build.gradle

This is the key part.  The build.gradle file will handle all of your dependencies including downloading your JDK and your JavaFX SDK so you don't even need to worry about that at all.  

### Directory and Package Structure

Gradle assumes a particular structure to you project, and StarterFX has just enough pieces that all of the structure is ready for you to use and build on without having to research where to put stuff.  

### A Separate Main Class

There's a quirk of JavaFX that it doesn't like to run from a Jar file where the main() method is in a class that extends Application.  So StarterFX splits it off for you, and the invokes then launch() method in the Application class properly.

### Enough Layout Code to Test That it Works

The Application class, StarterApplication, has a method that creates a Region that is added to a Scene which is loaded into the Stage which will put a window on your screen.  

### A Stylesheet Loaded From Resources

Trying to figure out how to reference a resource in your code can be mystifying.  StarterFX has a CSS file in its resources, which is loaded by the layout creation method.  You get code that not only loads the resource properly, but demonstrates how to install a stylesheet and how to use the selectors.

# How Do You Use It?

## Download as a Zip

Click on the "Code" button on the GitHub page, and select "Zip" file.  Open the Zip file and extract the contents to an empty directory on your computer.  

Highlight the "build.gradle" file and right click (or whatever mechanism your O/S uses) and select something like "Open with...".

Pick an application compatble with Gradle.  Intellij Idea, Eclipse and NetBeans should all work well.

Wait for your IDE to finish downloading the components and configuring the project.  Then select "Application -> Run" from the Gradle menu.  You should see some console activity, then the window should appear.

## Import Straight From GitHub in Your IDE

### From Intellij Idea

In the "File" menu, pick "New", then "Project From Version Control...".

You'll get a dialog box that looks a bit like this:

![GitHub Dialog]({{page.GitHubDialog}})

In the "URL" field you want to put this:

https://github.com/PragmaticCoding/StarterFX.git

And then specify a directory to place the project on your computer.  Click "Clone".

# What Next?

## Take a Look at build.gradle

The build.gradle file is going to tell Gradle how to assemble and compile the project, and where to find the code it needs to run the application.  The most important section is this:

``` java
sourceCompatibility = '16'
targetCompatibility = '16'


application {
    mainModule = 'ca.pragmaticcoding.starterfx'
    mainClass = 'ca.pragmaticcoding.starterfx.Main'
}

javafx {
    version = '16'
    modules = ['javafx.controls', 'javafx.graphics', 'javafx.base']
}
```

### The Versions

Everything in this project is set to run on JavaFX 16 and Java 16.  If you want to change to a different version, there are three places that you might need to make changes.

### The Packages

You can leave them as they are if you want, but if you intend on ever using your project as something other than a stand-alone application you'll probably want to change the packages from "ca.pragmaticcoding".  If you do change them, you'll need to change them in both the project structure and here, in the build.gradle.  If you change them only in one place, the project won't run any more.

### Modules

This is a modular project, which is the way that you should be creating your project nowadays.  If you change the package names, then you'll also need to update the module-info file.  If you don't do this, you'll get strange issues that will cause you pain and suffering.
