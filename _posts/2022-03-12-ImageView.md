---
title:  "All About Image and ImageView"
date:   2022-03-11 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/imageview
happy: /assets/elements/smileyface.png
sad: /assets/elements/sadface.png
broken: /assets/elements/broken.png
loading: /assets/elements/loading.gif
screensnap: /assets/elements/ImageViewDemo1.png
toc: true
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
excerpt: Let's look at Image and ImageView.  How they relate to each other, and how to use them.
---

# Introduction

`Image` and `ImageView` are the two basic classes that you'll need to master to be able put images into your layouts.  `ImageView` is the layout class, while `Image` is the "data" class, in much the same way that `String` is the data class for `Text`.  

# Image

`Image` is the "data" class for images.  You can load BMP, GIF, JPEG or PNG formatted images into an `Image` object.  

## How Image Fits in the JavaFX Hierarchy

`Image` inherits directly from `Object`.  It doesn't have any JavaFX parents.  However, it is still a JavaFX object and will require that the JavaFX engine is running.  It's also best to handle Images on the FXAT.

## How To Use Image

Every constructor for `Image` requires that image data, or the location of image data is specified.  There's no way to create an empty `Image` or to load image data after an `Image` has been instantiated.  In that respect, you may consider `Image` to be immutable.

There are two important points to remember about `Image`:

* A single `Image` can be placed into as many `ImageView` objects as you like.  This is very different from `ImageView` which can only be placed once into a layout.

* Loading images is generally one of the slowest processes in rendering a screen.  

What this means is that you *might* need to implement a scheme to avoid having the cost of of `Image` loading affect your GUI performance.  The simplest approach is to avoid loading the same `Image` resource over and over if it appears multiple times on a screen.  It might be possible to cache `Images` if they are to be dynamically displayed on the layout over time.  It also might be possible to treat a set of images as a `Sprite`, so the entire set is loaded into a single `Image` which is then reused.

## Creating an Image

There are two basic ways to load an image: from an `InputStream`, or from a URL.  Each method also allows you to scale the image if you want.  

### Background Loading

If the image is being loaded from a URL, you have the option of loading it on a background thread.  This is an important feature that doesn't seem to be used as much as it should be.  Since `Images` really should be handled on the FXAT, loading an `Image` from a remote location could cause blocking on the FXAT - which is a really, really bad thing.  

There are also properties that allow you to track the progress of the loading of an image on a background thread.  

## Scaling the Image

It's possible to scale an image when it is loaded in order to reduce its footprint in memory and to potentially avoid scaling calculations in the layout if the size needs to be modified in the `ImageView`.

## Error Handling

It's possible that your `Image` may fail to load.  In this case there are methods that can tell you if the load failed, and any exception that was thrown by the loading process.

## What Can You Do With an Image?

Not much, except put it into a screen object that holds `Image`.  This includes `BackgroundImage`, `ImageCursor`, `ImagePattern` and `BorderImage`.  We're going to stick to talking about just `ImageView` here.

# ImageView

`ImageView` is a `Node` type that you can add to a layout.  It will go anywhere that you can put `Node`.

## How ImageView Fits in the JavaFX Hierarchy

`ImageView` inherits methods and properties from just one JavaFX class:

Node
: Everything in JavaFX inherits from `Node`, which gives most of its methods for styling, event handling and responding to mouse actions as well as location and transformation.

## Instantiating ImageView

There are three ways to instantiate `ImageView`:

* Create an empty `ImageView`.

* Specify an image resource URL.  In this case JavaFX will automatically create an `Image` for you and load it from the URL, but it won't do background processing.

* Pass the constructor an `Image` object.

Unlike `Image`, `ImageViews` can have their contents changed, so you don't need to consider them immutable in this respect.

## Properties of ImageView

FitHeight, FitWidth and PreserveRatio
: These properties control the scaling of the ImageView on the screen to fit within defined parameters.

ViewPort
: This property defines a rectangle which allows only a portion of an image to be visible in the ImageView.

Image
: This is the Image contained in the ImageView.


## Uses of ImageView

Of course, `ImageView` can be used on its own to put an image onto the layout somewhere.  Something like this:

``` java
public class ImageViewDemo1 extends Application {

    public static void main(String[] args) {
        launch();
    }

    @Override
    public void start(Stage stage) throws IOException {
        Scene scene = new Scene(createContent(), 820, 640);
        stage.setScene(scene);
        stage.show();
    }

    private Region createContent() {
        Image loadingGif = new Image(Objects.requireNonNull(ImageViewDemo1.class.getResource("loading.gif")).toString());
        Image brokenImage = new Image(Objects.requireNonNull(ImageViewDemo1.class.getResource("broken.png")).toString());
        ImageView imageView = new ImageView(loadingGif);
        imageView.setImage(loadingGif);
        Image image = new Image("https://www.pragmaticcoding.ca/assets/images/794.png", 700, 0, true, true, true);
        image.progressProperty().addListener(observable -> {
            System.out.println("Progress " + image.getProgress());
            if (image.getProgress() == 1.0) {
                if (!image.isError()) {
                    imageView.setImage(image);
                } else {
                    imageView.setImage(brokenImage);
                }
            }
        });
        VBox vBox = new VBox(imageView);
        vBox.setPadding(new Insets(30));
        return vBox;
    }
}
```
This program demonstrates how to use the background loading of `Image` with a large file.  First, we create two `Images` from local resources; one for "In Progress", which is an animated GIF and the other is to show a broken link.  Then we create the `ImageView` and load it up with the "In Progress" GIF, so we'll see that on the screen immediately.  

Then we create the `Image` that we really want to show on the screen.  But it's coming from a remote site, and it's pretty big (about 5MB), so we'll load it in the background.  Since it's a remote file, we really do need to worry about a problem with the file, so we are going to put some handling in there to detect a problem and put up the "Broken Link" image.

In order to know when the `Image` has finished loading, we need to place a listener on the `Progress` property of the `Image`.  The `Progress` is a `Double` that runs from 0 to 1.0.  Unfortunately, a call to `image.isBackground()` always returns `true`, even after the image has finished loading, so we cannot use it test for load completion.

In the listener, we check for the progress to reach "1.0", then check to see if it's encountered an error and then populate the `ImageView` with either the freshly loaded `Image`, or the "Broken" `Image`.

This is typical JavaFX.  All the tools are provided for you to build a slick, professional UI but they're all relatively low level.  This can result in a lot of "boilerplate" code.  If you find yourself repeating this pattern over and over, it might be worth creating a custom class that handles all of this in one place.  Then you can reuse it over and over.

#### What About the "Placeholder" Image

If you really search through the JavaDocs for `Image` with eagle-eyes, you'll find the following intriguing piece of information in the sample code in the description:

``` java
// load an image in background, displaying a placeholder while it's loading
// (assuming there's an ImageView node somewhere displaying this image)
// The image is located in default package of the classpath
Image image1 = new Image("/flower.png", true);
```
And then nothing more about "a placeholder" anywhere else in the JavaDocs.

I've looked through the source code for JavaFX and I couldn't find any reference to a placeholder anywhere.  Of course the code for `Image` is super complicated, since it has all kinds of different formats and threading and graphical stuff going on, so I could have missed it.  My guess is that this an "unimplemented feature" that never got off the ground.  It's a shame, because it would have been so much easier to just call a `setPlaceholder()` method than to go through the listener route.

In the meantime, if you just put an `Image` that's still loading into an `ImageView`, you'll just get a blank.


### As Part of Labeled

Probably the most common use of `ImageView` is as the `Graphic` in any of the classes that inherit from `Labeled`.  This includes, `Label`, `Button`, and the various `Cell` classes.  

In this example, we're going to put the `ImageViews` into `Buttons`:

``` javaScreenSnap
public class ImageViewDemo2 extends Application {

    private final ImageView imageView = new ImageView();

    public static void main(String[] args) {
        launch();
    }

    @Override
    public void start(Stage stage) {
        Scene scene = new Scene(createContent(), 500, 320);
        stage.setScene(scene);
        stage.show();
    }

    private Region createContent() {
        Image happyFace = new Image(Objects.requireNonNull(ImageViewDemo2.class.getResource("smileyface.png")).toString());
        Image angryFace = new Image(Objects.requireNonNull(ImageViewDemo2.class.getResource("sadface.png")).toString());
        HBox hBox = new HBox(createButton("Loved it!", happyFace),
                createButton("Liked it!", happyFace),
                createButton("Hated it!", angryFace));
        hBox.setPadding(new Insets(30));
        imageView.setPreserveRatio(true);
        imageView.setFitHeight(160);
        return new VBox(20, hBox, imageView);
    }

    private Button createButton(String text, Image image) {
        Button results = new Button(text, createButonImageView(image));
        results.setOnAction(evt -> imageView.setImage(image));
        return results;
    }

    private Node createButtonImageView(Image image) {
        ImageView results = new ImageView(image);
        results.setSmooth(true);
        results.setPreserveRatio(true);
        results.setFitHeight(64);
        return results;
    }
}
```
And it looks like this:

![ScreenSnap]({{page.screensnap}})

A few things to note here:

* The `happyFace` `Image` is used twice in the `Buttons`, and again in the `ImageView` below the `Buttons`.  This doesn't cause any problems.
* The `Images` are all loaded in the foreground.  There would be no harm in loading them in the background, and probably should be.  However, with just two reasonably small images to load, there's probably almost no impact on the UI performance.
* You can freely swap out the `Image` in an `ImageView` at any time while the application is running and it works just fine.  This suggests the best way to handle layouts with variable images in them.

# Images For the Examples

![Loading]({{page.loading}}) ![Broken]({{page.broken}}) ![Happy]({{page.happy}})

![Angry]({{page.sad}})
