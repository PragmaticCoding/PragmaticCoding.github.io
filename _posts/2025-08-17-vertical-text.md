---
title:  "Vertical Text"
date:   2025-08-18 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/vertical-text
ScreenSnap0: /assets/elements/VerticalText0.png
ScreenSnap1: /assets/elements/VerticalText1.png
ScreenSnap2: /assets/elements/VerticalText2.png
ScreenSnap3: /assets/elements/VerticalText3.png
ScreenSnap4: /assets/elements/VerticalText4.png
ScreenSnap5: /assets/elements/VerticalText5.png
ScreenSnap6: /assets/elements/VerticalText6.png
ScreenSnap7: /assets/elements/VerticalText7.png
ScreenSnap8: /assets/elements/VerticalText8.png
ScreenSnap9: /assets/elements/VerticalText9.png
ScreenSnap10: /assets/elements/VerticalText10.png

Bounds: /assets/elements/boundsParent.png
Diagram: /assets/elements/VerticalTextTranslate.png
OLArticle: /javafx/elements/observable-classes-lists
OBGuide: /javafx/elements/observables_guide
GitHubLink: https://github.com/thomasnield/VerticalText

JavaDocOL: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/ObservableList.html
JavaDocFXC: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/FXCollections.html

excerpt: "What if you want to have the text in a Node go up (or down) the screen instead of from left to right?  This can be difficult because the Text class doesn't natively support it.  This article will show you how to do it."
---

# Introduction

This article was inspired by a question posted on the JavaFX subreddit on Reddit.com.  It seemed to be too complicated a subject to just answer in the thread, so I decided to write an entire article around it and explain the whole subject properly.

I've whipped this up super fast, in the hopes that it will still be relevant to the user that posted the question 4 days ago now.

# The Post From Reddit

I'm going to reproduce the entire original post because it does describe the problem very clearly...

## TableView is just an absolutely horrible class and UI control.

I have a lot of numeric columns I want to show. It makes perfect sense to have the text in the headers be rotated 90 degrees so that the columns don't have to be wide. But all of this is a mess...

* if you replace the text with a label that is rotated 90 degrees as the TableColumn graphic then all of the following problems occur.

  *  Rotated labels are stupid in JavaFX. It doesn't just rotate the text, it rotates the label and "width" and "height" of the Label node basically become meaningless.

  *  Even if you wrap it in a Pane or something and recompute the bounds the header doesn't resize.

  *  The headers of columns don't share heights so no header resizing is done anyways.

* There's some sort of "Skin" method that allows the headers to resize but this is just stupid because you're diving way too deep into the bowels of layout that you shouldn't have to do for a UI concept that is this simple.

  * Positioning becomes absolute nonsense, BOTTOM_LEFT moves the graphic too low and clips it.

  * If all the labels aren't the same size then the resizing of the header doesn't align them, they all get centered.

There should just be a way to tell the header to rotate its text or graphic. And all the headers of all the columns that are added to the same TableView should share the same height. I just can't image a Table with columns with different header heights.

Summary: I shouldn't have to write a hundred lines of code to get vertical header labels in a table.

If you know an easier way... please, enlighten me.

## The Problem Isn't TableView

Before we go any further, it's true that `TableView` is extremely complicated and can be difficult at times to figure out just what you need to do in order to style it the way that you want.

But in this particular case, `TableView` and its complexity is **not** the problem.  It's actually `Text` that is causing difficulties.  

That's because you don't need (and the OP pointed out that it's "just stupid") to mess with the inner workings of `TableColumnHeader`, what you need to deal with is the stuff that you *put into* `TableColumnHeader`.  And, just as with `Label`, you can have both a `text` element and a `graphic` element in a `TableColumnHeader`.  There's not much we can do with the `text` element, but the `graphic` element allows us to put any kind of `Node` that we want into the `TableColumnHeader`.  That's the key to solving the problem.

The rest of this article is going to focus on how create a `Text` that runs up the screen instead of along to the right, and to get it to play nicely with the layout.  Then, at the end, we'll stuff it into a `TableView` and see how it works.

# Things We Need to Figure Out

Before we do anything else, you should know that you can do this:

``` kotlin
   val text = Text("This is some text").apply {
     rotate = 270.0
   }
```
And it *will* work.  But the layout manager will treat it was still taking up space the same way it was before it was rotated.     

This means that there are two issues that we need to address:

1. `Text` is designed to display horizontally, with no provision for vertical display.  `Text` itself is super complicated and would be difficult to extend and modify to display vertically.
1. Transformations applied to `Nodes` are largely ignored by the layout manager.

We can deal with issue #1 by just ignoring it.  There's no way that we are going to customize it so we'll just apply a rotation and then deal with the layout issues from item #2.

We can deal with #2 by putting our `Text` into some kind of container class, and then letting the layout manager use the size of the container instead of the `Text`.

# Putting the Text into a StackPane

`StackPane` is an excellent general purpose container class that will also keep its contents centred for us.  So we'll start there.  Here's a little application that will demonstrate what we can do:

``` kotlin
class VerticalTextExample0() : Application() {
    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(), 500.0, 500.0)
        stage.show()
    }

    private fun createContent(): Region = HBox(5.0).apply {
        children += StackPane().apply {
            val text = Text("This is some text").apply {
                rotate = 270.0
            }
            children += text
            maxWidth = 3.0
            border = Border(BorderStroke(Color.RED, BorderStrokeStyle.SOLID, null, null))
        }
        border = Border(BorderStroke(Color.GREEN, BorderStrokeStyle.SOLID, null, null))
        padding = Insets(20.0)
    }

}

fun main() = Application.launch(VerticalTextExample0::class.java)
```
The main part of the screen is an `HBox` with a green border, and it contains a `StackPane` with a red border, which, in turn contains a `Text` which has been rotated 270 degrees.  Note that the `maxWidth` of the `StackPane` has been set to 3 pixels.  

It looks like this:

![Screen Snap]({{page.ScreenSnap0}})

The `Text` is there, and it's rotated and it's still in the centre of the `StackPane`.  This is all good.

You can see that the width of the `StackPane` isn't 3 pixels, though.  In fact, it's exactly the same size that it would need to be to hold the `Text` if it was still horizontal:

![Screen Snap]({{page.ScreenSnap1}})

I've just added a second `Text` with the same message so that it's clear that the width of the `StackPane` isn't arbitrary.

Since it won't let us shrink the StackPane any more, we'll have to try a container class that will:  `Pane`.

# Putting the Text into a Pane

`Pane` is generally not a very good container class, and you shouldn't use it unless you have some very specific needs that can't be met any other way.  We do, so we *will* use it:

``` kotlin
private fun createContent(): Region = HBox(5.0).apply {
    children += Pane().apply {
        val text = Text("This is some text")
        children += text
        border = Border(BorderStroke(Color.RED, BorderStrokeStyle.SOLID, null, null))
    }
    border = Border(BorderStroke(Color.GREEN, BorderStrokeStyle.SOLID, null, null))
    padding = Insets(20.0)
}
```
And it looks like this:

![Screen Snap]({{page.ScreenSnap2}})

This is a bit goofy looking.  The `Text` is outside the `Pane`!  That's because the origin for `Text` is normally the "baseline" which is the bottom of the first line of text.  This is one of the reasons that you should stay away from `Pane`, because it almost always requires translation of your `Nodes` to make it work - and that's usually a bad thing.  

Before we go any further, we'll mess with the width of our `Pane` just to ensure that we can make it small, and we'll change the origin of the `Text` so that it's inside the `Pane`.  I've also added a second `Pane` with the same `Text` to the `HBox` so that we can see how the layout manager uses the space:

``` kotlin
private fun createContent(): Region = HBox(5.0).apply {
    children += Pane().apply {
        val text = Text("This is some text").apply {
            textOrigin = VPos.TOP
        }
        children += text
        border = Border(BorderStroke(Color.RED, BorderStrokeStyle.SOLID, null, null))
        maxWidth = 50.0
    }
    children += Pane().apply {
        val text = Text("This is some text").apply {
            textOrigin = VPos.TOP
        }
        children += text
        border = Border(BorderStroke(Color.BLUE, BorderStrokeStyle.SOLID, null, null))
    }
    border = Border(BorderStroke(Color.GREEN, BorderStrokeStyle.SOLID, null, null))
    padding = Insets(20.0)
}
```
It looks like this:

![Screen Snap]({{page.ScreenSnap3}})

A bit ugly, but you can see that `Pane` doesn't care about having its contents exceed its width, and the layout manager uses the `Pane` width, and not the contents width.

Now, let's rotate that text.  First without the `maxWidth` on the `Pane`:

![Screen Snap]({{page.ScreenSnap4}})

Just a side note
:  I've chosen to rotate the `Text` 270 degrees, which makes in go up, because that looks right to me.  However, if I look at my bookcase, I see that all my books go *down*, which also looks correct to me.  However, I know that French books go *up* and that looks very weird to me.  But having text on the screen go down looks weird - to me, at least.  You pick what works for you, just change the `270.0` to `90.0`.

And if we put a `maxWidth = 25.0`:

![Screen Snap]({{page.ScreenSnap5}})

It's pretty clear from this that we are on the right track.  All we need to do is figure out how to move the `Text` into the right place, and how to size the `Pane` correctly.

# Using Bounds

The first issue we have is that `Text` doesn't have a `height` or `width` property. However, all `Nodes` have a concept called "bounds", which is a rectangular box around their contents.

If only it were that simple.

There are a few different contexts for bounds, and that usually doesn't matter unless you are applying transformations.  Of course, we already applied one transformation - the rotation - and now we are going to do some translations as well.  So we need to understand bounds a little bit better.

There are two contexts that we care about;

Bounds in Local
: This is the bounding box around the `Node` from the perspective of the `Node` itself.  If you move or rotate the `Node`, this bounding box won't change from the perspective of the `Node`.

Bounds in Parent
: This is the bounding box around the `Node` from the perspective of its container.  This is what you'd see when looking at it on the screen.

The JavaDocs for `Node` has a diagram that illustrates this:

![Bounding Box]({{page.Bounds}})

The sine wave was originally horizontal and green box shows the bounds from the perspective of the sine wave.  The red box shows the bounds from the perspective of the container after the sine wave has been rotated.

Every `Node` has a `Property` called `boundsInParent`, and another called `boundsInLocal`.  These are of the type `Bounds` which will tell us the location of the centre of the box, the height of the box and the width of the box.

Let's see how these work:

``` kotlin
val text = Text("This is some text").apply {
    textOrigin = VPos.TOP
    boundsInParentProperty().subscribe { newValue ->
        with(newValue) {
            println("Parent Min Y: $minY\t\tMin X: $minX")
            println("Parent Max Y: $maxY\t\tMax X: $maxX")
            println("Parent Centre Y: $centerY\t\tCentre X: $centerX")
            println("Parent Height: $height\t\tWidth: $width")
            println()
        }
    }
    boundsInLocalProperty().subscribe { newValue ->
        with(newValue) {
            println("Local Min Y: $minY\t\tMin X: $minX")
            println("Local Max Y: $maxY\t\t\tMax X: $maxX")
            println("Local Centre Y: $centerY\t\tCentre X: $centerX")
            println("Local Height: $height\t\t\tWidth: $width")
            println()
        }
    }
    println("Rotating")
    rotate = 270.0
}
```
In this code, we add some `Subscriptions` (which are just like `ChangeListeners`, only better) to these two `Properties` so that we can see how they respond to transformations.

When run, it looks exactly the same as the last screen shot, but now we get this console output:

```
> Task :run
Parent Min Y: 0.0                   Min X: 0.0
Parent Max Y: 17.70600128173828     Max X: 101.73800659179688
Parent Centre Y: 8.85300064086914   Centre X: 50.86900329589844
Parent Height: 17.70600128173828    Width: 101.73800659179688

Local Min Y: 0.0                    Min X: 0.0
Local Max Y: 17.70600128173828      Max X: 101.73800659179688
Local Centre Y: 8.85300064086914    Centre X: 50.86900329589844
Local Height: 17.70600128173828     Width: 101.73800659179688

Rotating
Parent Min Y: -42.0160026550293     Min X: 42.0160026550293
Parent Max Y: 59.72200393676758     Max X: 59.72200393676758
Parent Centre Y: 8.85300064086914   Centre X: 50.86900329589844
Parent Height: 101.73800659179688   Width: 17.70600128173828
```
The version of `Subscription` that I used automatically fires as soon as it is created.  The first two sets of output are from those initial firings, one on each `Property`.  The third set of output comes from `boundsInParent` after the rotation has been performed.  You can see the "Rotating" output that came from the `println()` right before the rotation was applied.

Things to notice:

1. The `boundsInLocal` did not change after the rotation, but the `boundsInParent` did.
1. The two data sets are identical before the rotation.
1. The centre X/Y did not change after the rotation because the rotation *was around the centre*.
1. The height and width flipped after the rotation, but their magnitudes did not change.
1. The "min" and "max" numbers are all over the place.

One last thing to be aware of is that at least some of this behaviour is due to fact that we rotated an odd multiple of 90 degrees.  Things would be different at any other angles.  But 270 degrees is what we want (or 90 degrees if you want the text to read down, instead of up), so we're good.

Now that we know the size and the centre of our `Text` we can look into moving it to the correct place.

# Translating the Text

Where do we want the `Text`?

The `Pane` is always going to occupy a rectangular area going down and to the right from its (0,0) or top-left corner.  We need to have the top-left corner of our `Text` line up with the top-left corner of the `Pane`.  But directly thinking about that top left corner will have you looking at those "min X" and "min Y" values, and we know that they are a bit crazy after the rotation.

The centre, however, is easier to deal with.

Currently that centre is located 1/2 the length of the `Text` along the X-axis and 1/2 the height of the `Text` down the Y-axis.  We need to move it such that it is 1/2 the length of the `Text` down the Y-axis, and 1/2 the height of the `Text` along the X-axis.  

But we cannot just specify this new location directly for the transform.  We need to specify the *difference* between these two locations:

![Diagram]({{page.Diagram}})

If we use `boundsInLocal.centerX` and `boundsInLocal.centerY` instead of "(1/2 Length, 1/2 Height)" for the starting point then we don't need to worry about where the `Text` actually starts out, it will always calculate the correct translation to put it in the top left corner of the `Pane`.  We could even skip the `textOrigin` adjustment, because this method would always compensate automatically.

We need two translations, one for "X" and one for "Y":

``` kotlin
translateY = (boundsInLocal.width / 2) - boundsInLocal.centerY
translateX = (boundsInLocal.height / 2) - boundsInLocal.centerX
```
This will position it correctly.

# Sizing the Pane

The last step is to size the `Pane` so that it just fits around the `Text` in its translated location.  This is the easiest part:

``` kotlin
maxWidth = newValue.height
maxHeight = newValue.width
minWidth = newValue.height
minHeight = newValue.width
```
We can't actually set `Pane.width` and `Pane.height` because they are read-only.  However, since `Pane` is fairly free-form with regards to its sizing, we can effectively set its width and height by specifying the maximum and minimum values.  

# Dealing with Changes to the Text

Clearly, all of these calculations are dependent on the location of the centre of the `Text` according to `boundsInLocal`, and the width and height of the `Text` according to `boundsInLocal`.  These values themselves, however, are totally dependent on the content of the `Text`.  This means the actual `String` value of the text, along with the font and the font size.  If those things change, then `boundsInLocal` is likely to change too.  

When `boundsInLocal` changes, then all of our calculations need to change as well.  This means that we need to put them in a `Subscription` on `boundsInLocalProperty()`.  Our final layout code looks like this:

``` kotlin
private fun createContent(): Region = HBox(5.0).apply {
    children += Pane().apply {
        val text = Text("This is some text").apply {
            textOrigin = VPos.TOP
            boundsInLocalProperty().subscribe { newValue ->
                with(newValue) {
                    translateY = (boundsInLocal.width / 2) - boundsInLocal.centerY
                    translateX = (boundsInLocal.height / 2) - boundsInLocal.centerX
                }
            }
            println("Rotating")
            rotate = 270.0
        }
        children += text
        text.boundsInLocalProperty().subscribe { newValue ->
            maxWidth = newValue.height
            maxHeight = newValue.width
            minWidth = newValue.height
            minHeight = newValue.width
        }
        maxWidth = 25.0
        border = Border(BorderStroke(Color.RED, BorderStrokeStyle.SOLID, null, null))
    }
    children += Pane().apply {
        val text = Text("This is some text").apply {
            textOrigin = VPos.TOP
        }
        children += text
        border = Border(BorderStroke(Color.BLUE, BorderStrokeStyle.SOLID, null, null))
    }
    border = Border(BorderStroke(Color.GREEN, BorderStrokeStyle.SOLID, null, null))
    padding = Insets(80.0)
}
```
And it looks like this:

![Screen Snap]({{page.ScreenSnap6}})

If we make the `Text` content bigger with multi-lines, we get this:

![Screen Snap]({{page.ScreenSnap7}})

Since this is based on `boundsInLocal` as a `Property` any changes to the `Text` content are going to automatically readjust in real time.

# Creating a Custom Control

The only thing left is to set this up as a custom `Control` that we can use, just like a `Text` (well, nearly like) and put into our layouts without any fuss.

For this, we'll convert from `Pane` to `Region` which will prevent client code from mucking about with its contents:

``` kotlin
class VerticalText(initialValue: String) : Region() {

    val verticalText = Text(initialValue).apply {
        styleClass += "text"
        textOrigin = VPos.TOP
        boundsInLocalProperty().subscribe { newValue ->
            with(newValue) {
                translateY = (boundsInLocal.width / 2) - boundsInLocal.centerY
                translateX = (boundsInLocal.height / 2) - boundsInLocal.centerX
                this@VerticalText.maxWidth = newValue.height
                this@VerticalText.maxHeight = newValue.width
                this@VerticalText.minWidth = newValue.height
                this@VerticalText.minHeight = newValue.width
            }
        }
        rotate = 270.0
    }

    fun textProperty(): StringProperty = verticalText.textProperty()
    fun setText(newValue: String) {
        verticalText.text = newValue
    }

    fun getText(): String = verticalText.text

    init {
        styleClass += "vertical-text"
        children += verticalText
        border = Border(BorderStroke(Color.RED, BorderStrokeStyle.SOLID, null, null))
    }
}
```
This is pretty small and simple.  It's just like our layout version except that it's a class of its own.  There's only one `Subscription` because we can access the class `maxHeight`, `maxWidth`, `minWidth` and `minHeight` properties from inside the `verticalText` field.  The class has the Stylesheet selector "vertical-text", and the `Text` has the selector "text" which will become "vertical-text .text" in the Stylesheet.

The `textProperty()` of the embedded `Text` is exposed to the client code via a set of functions that delegate to that embedded `Text`.  The `Text` itself is public, but is still a `val`.  This means that client code can get to it to change the font and other `Properties`, but it cannot be replaced by the client code.

# Integrating into TableView

The final step of answering the original question on Reddit is to integrate this new `VerticalText` into the column heads of a TableView.  Let's look at how this would work:

``` kotlin
private fun createContent(): Region = HBox(5.0).apply {
    val listItems = FXCollections.observableArrayList<DataModel>()
    listItems += DataModel("Fred Brown", 2, 7, "Some More Text")
    listItems += DataModel("Fred Brown", 2, 7, "Some More Text")
    listItems += DataModel("Fred Brown", 2, 7, "Some More Text")
    listItems += DataModel("Fred Brown", 2, 7, "Some More Text")
    listItems += DataModel("Fred Brown", 2, 7, "Some More Text")
    listItems += DataModel("Fred Brown", 2, 7, "Some More Text")
    children += TableView<DataModel>().apply {
        columns += TableColumn<DataModel, String>("Name").apply {
            cellValueFactory = Callback { p -> p.value.name }
        }
        columns += TableColumn<DataModel, Int>().apply {
            graphic = VerticalText("Value 1")
            cellValueFactory = Callback { p -> p.value.val1.asObject() }
        }
        columns += TableColumn<DataModel, Int>().apply {
            graphic = VerticalText("Second Value")
            cellValueFactory = Callback { p -> p.value.val2.asObject() }
        }
        columns += TableColumn<DataModel, String>("Suffix").apply {
            cellValueFactory = Callback { p -> p.value.suffix }
        }
        items = listItems
    }
    border = Border(BorderStroke(Color.GREEN, BorderStrokeStyle.SOLID, null, null))
    padding = Insets(80.0)
}
```
Here's `DataModel`:

``` kotlin
class DataModel(initName: String, initVal1: Int, initVal2: Int, initSuffix: String) {
    val name: StringProperty = SimpleStringProperty(initName)
    val val1: IntegerProperty = SimpleIntegerProperty(initVal1)
    val val2: IntegerProperty = SimpleIntegerProperty(initVal2)
    val suffix: StringProperty = SimpleStringProperty(initSuffix)
}
```
And it looks like this:

![Screen Snap]({{page.ScreenSnap8}})

The `VerticalText` for `Second Value` is pushing up against the borders of the `ColumnHeading` so we can add some spacing by putting it inside a `StackPane`:

``` kotlin
graphic = StackPane(VerticalText("Second Value")).apply {
    padding = Insets(5.0, 0.0, 5.0, 0.0)
}
```
And now it looks like this:

![Screen Snap]({{page.ScreenSnap9}})

Which is fine.  I've also manually shrunk the width the "Second Value" column to show how it doesn't occupy any extra space.

If you find that you're doing this padding trick a lot, you could just bake it into your own version of `VerticalText` so that you don't have to worry about it.  You could even do it by adjusting the translation to account for padding, and then making the size of the `Pane` itself bigger.

# Conclusion

There you go.  We have a 30 line class that actually goes way beyond the original brief, as it includes styling selectors and the ability to bind the `textProperty()` of the enclosed `Text` directly, as well as access its `Properties` via the `Text` itself.  In other words, it's a fully functional custom `Node` that you can use just about anywhere.

With JavaFX, any time you find yourself feeling something like this:

> ...is just stupid because you're diving way too deep into the bowels of layout that you shouldn't have to do for a UI concept that is this simple.

That's a really good indicator that you're approaching the problem from the wrong direction.  Not always, but probably.

The real issue is that solving this problem, even though it's actually quite simple, requires a pretty deep understanding of some non-beginner concepts in order to know the correct approach.  

When I approached this problem, there were some things that I didn't know along with some things that suspected might not work.  For instance, I *was* hoping that just rotating inside `StackPane` and adjusting `maxWidth` would the solution.  I suspected that it wouldn't work.  When it didn't work, I knew that `Pane` was going to be the answer.

I was mildly surprised that `Text` doesn't have width or height `Properties`.  I did remember about `Bounds` though, and it only took a few seconds to locate it in `Node`. Most of the investigative/learning time was spent understanding how the "local" vs "parent" contexts worked and how to get the translations correct.

The bottom line is that I have enough experience with JavaFX that I know what I don't know.  And when you know that, you know where to go looking for the right answers.

Back to the OP:

> Summary: I shouldn't have to write a hundred lines of code to get vertical header labels in a table.

No you shouldn't.  And you don't.
