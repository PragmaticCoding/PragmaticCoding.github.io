---
title:  "Bad Advice"
date:   2025-06-12 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/wrapper-mistake
PantsOnFire: /assets/images/PantsOnFire.png
ScreenSnap0: /assets/elements/TransformList0.png
ScreenSnap1: /assets/elements/TransformList1.png
ScreenSnap2: /assets/elements/TransformList2.png
ScreenSnap3: /assets/elements/TransformList3.png
ScreenSnap4: /assets/elements/TransformList4.png
ScreenSnap5: /assets/elements/TransformList5.png
ScreenSnap6: /assets/elements/TransformList6.png
ScreenSnap7: /assets/elements/TransformList7.png
ScreenSnap8: /assets/elements/TransformList8.png
ScreenSnap9: /assets/elements/TransformList9.png
ScreenSnap10: /assets/elements/TransformList10.png

Diagram: /assets/elements/ListProperties.png
OLArticle: /javafx/elements/observable-classes-lists
OBGuide: /javafx/elements/observables_guide
ROGuide: /javafx/elements/observable-classes-generics#readonlyobjectpropertybase-and-readonlyobjectwrapper

JavaDocOL: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/ObservableList.html
JavaDocFXC: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/FXCollections.html

excerpt: "The ReadOnlyObjectWrapper"
---

# Introduction

https://solutionfall.com/question/what-is-the-purpose-of-using-readonlyobjectwrapper-in-tablecolumns-setcellvaluefactory-method/

>This approach assumes that the object returned from p.getValue() has a JavaFX ObservableValue that can simply be returned. The benefit of this is that the TableView will internally create bindings to ensure that, should the returned ObservableValue change, the cell contents will be automatically refreshed.
>
>In situations where a TableColumn must interact with classes created before JavaFX, or that generally do not wish to use JavaFX apis for properties, it is possible to wrap the returned value in a ReadOnlyObjectWrapper instance. For example:
>
>
``` java
 firstNameCol.setCellValueFactory(new Callback<CellDataFeatures<Person, String>, ObservableValue<String>>() {
     public ObservableValue<String> call(CellDataFeatures<Person, String> p) {
         return new ReadOnlyObjectWrapper(p.getValue().getFirstName());
     }
  });
```

I'll repeat what it says in the text, "...it is possible to wrap the returned value in a ReadOnlyObjectWrapper instance...", because this emphasizes that this insn't some typo or mistake in the attached code sample.  

And, if you are like me, when you read this you get the impression that `ReadOnlyObjectWrapper` is some kind of utility class that allows you to turn a non-`Observable` data element into an `Observable` for the sake of compatibility.  

In other words, any time you need to pass an `Observable`, but all you have is a primitive, stuff it into `ReadOnlyObjectWrapper` and off you go.  Furthermore, this is what `ReadOnlyObjectWrapper` is all about.

# What is `ReadOnlyObjectWrapper`?

That turns out to be a really good question. It's not at all what I thought...

I learned all of this when I wrote my ["Complete Guide to the Observable Classes"]({{page.OBGuide}}), and explained it in detail [here]({{page.ROGuide}}).  

Many times, when you want to expose a `Property` inside a class that you don't want client code to update, you'll return that `Property` as an `ObservableValue`, like this:

``` kotlin
class SomeClass() {

  private val someProperty : IntegerProperty = SimpleIntegerProperty(0)


  fun someProperty() : ObservableIntegerValue = someProperty

}
```
When client code calls `SomeClass.someProperty()`, then they'll get an `ObservableIntegerValue` and not an `IntegerProperty`.  This means that they will not be able to invoke `someProperty.set()` because this is not included in the `ObservableIntegerValue` interface.  

But there is a loophole.  There is no way to instantiate an `ObservableIntegerValue` because it is *just* an interface.  The only concrete classes that you can actually instantiate out of the entire class/interface structure is `SimpleIntegerProperty` and its subclasse `ReadOnlyIntegerWrapper`.  You can also extend `IntegerBinding` and create a custom subclass that you could instantiate, and that would also implement `ObservableIntegerValue`.

Now, what this means is that any `ObservableIntegerValue` created via the method in the code snippet above is going to actually be instantiated as `SimpleIntegerProperty`.  This makes sense when you think about it.  What good is an `Observable` that can truly never change?  No `Listeners` could ever fire and it would be no better than any generic wrapper class.  

But this means that any `ObservableIntegerValue` can be cast to at least `IntegerProperty` by client code.  And, of course, `IntegerProperty` supports `IntegerProperty.set()`.  Like this:

``` kotlin
val x: ObservableIntegerValue = SimpleIntegerProperty(0)
println("Is SimpleIntegerProperty: ${x is SimpleIntegerProperty}")
(x as IntegerProperty).set(5)
```
This code will run, will print out "Is SimpleIntegerProperty: true", and will change the value to 5 without error.

But what if you have some implementation where it is really, really, really important that `IntegerProperty.set()` is never available?

Enter `ReadOnlyIntegerProperty` and `ReadOnlyIntegerWrapper`.  These two classes together solve the enigma of having a truly read-only `Observable` that, at the same time, might have its value change and fire `Listeners`.

First off, `ReadOnlyIntegerWrapper` extends the concrete class `SimpleIntegerProperty` and adds a single method `getReadOnlyProperty()`.  If you look at the JavaDocs for `ReadOnlyIntegerWrapper` you'll see this:

> This class provides a convenient class to define read-only properties. It creates two properties that are synchronized. One property is read-only and can be passed to external users. The other property is read- and writable and should be used internally only.

It says that the two properties are "synchronized", but they aren't really.  The `get()` method of the `ReadOnlyIntegerProperty` that is returned from `getReadOnlyProperty()` actually delegates to the originating `ReadOnlyIntegerWrapper`.  In fact, the `ReadOnlyIntegerProperty` doesn't actually have a `value` at all!  You can add `Listeners` to a `ReadOnlyIntegerProperty`, but they are actually fired by the originating `ReadOnlyIntegerWrapper` when its `value` is invalidated.  

Finally, `ReadOnlyIntegerProperty` isn't implemented by any classes that have a `value` or that support `set()`.  This means that there is no way to cast a `ReadOnlyIntegerProperty` to any class that does support `set()`.  It is truly read-only, and yet will still behave like an `Observable`.

The `ReadOnlyIntegerWrapper`, however, can be treated just like an `IntegerProperty` or `SimpleObjectProperty`.  In fact, you could, if you wanted, use `ReadOnlyIntegerWrapper` instead of `SimpleIntegerProperty` everywhere in your applications and it wouldn't make any difference to how they work.

# Back to the JavaDocs for TableColumn

By the same token, however, if you never call `ReadOnlyIntegerWrapper.getReadOnlyProperty()`, then you might as well just use `SimpleIntegerProperty`.

Now, when you look at the example code from the JavaDocs again:

``` java
 firstNameCol.setCellValueFactory(new Callback<CellDataFeatures<Person, String>, ObservableValue<String>>() {
     public ObservableValue<String> call(CellDataFeatures<Person, String> p) {
         return new ReadOnlyObjectWrapper(p.getValue().getFirstName());
     }
  });
```
You can see that they never call `ReadOnlyObjectWrapper.getReadOnlyProperty()`.  And that value is returned as an `ObservableValue<String>`, so `getReadOnlyProperty()` isn't going to be available to whatever calls this `CellValueFactor` (unless it casts it to `ReadOnlyObjectWrapper`).  

This means that they might as well have just returned `SimpleObjectProperty` here instead.  But they didn't.

Let's not forget that this `Callback` is only ever going to be invoked by some internal code in `TableView` or `TableColumn`.  I think that we can safely assume that whatever code that is, it's not going to cast the results to `ObjectProperty` and then call `set()`.  So there is no value at all to going through the process of creating `ReadOnlyObjectWrapper` in order to return a `ReadOnlyObjectProperty`.

If we go back to that explanatory text,

> ...it is possible to wrap the returned value in a ReadOnlyObjectWrapper...

we see that this is a deliberate choice. What were they thinking?

I strongly suspect that most programmers that read this explanation (and probably the person that wrote it), view `ReadOnlyObjectWrapper` as something that works like this:

``` kotlin
class ImmutableStringValue(private val immutableValue: String?) : ObservableValue<String> {
    override fun addListener(listener: ChangeListener<in String>?) {}

    override fun removeListener(listener: ChangeListener<in String>?) {}

    override fun getValue(): String? = immutableValue

    override fun addListener(listener: InvalidationListener?) {}

    override fun removeListener(listener: InvalidationListener?) {}
}
```
This is clearly a "wrapper" around an immutable value, and this wrapper behaves *exactly* as you would expect an `ObservableValue<String>` to behave.  There's no point in keeping track of the `Listeners` that are added, or to actually try to remove them because they can **never** fire.  

This class does what appears to be needed.  Our source data isn't actually `Observable`, so there's no value in really implementing `Observable` methods.  However, it does provide type compatibility with `ObservableValue` and there is zero possibility that using this would ever "break" any client code that received it when expecting a "real" `ObservableValue`.

I think that a lot of programmers obssess about a percieved "overhead" when using `Observables`.  This is especially true when they have large amounts of data, and worry about excessive memory usage and performance issues when processing large amounts of `Obseravble` data.  

My general sense from looking at the internal code to see how these classes actually work, they are designed to minimize memory and performance overhead, and that an `Observable` without any `Listeners` added won't even have data storage allocated for `Listeners` until the first one is added.  There is code that checks for `Listeners` whenever an `ObservableValue` is changed, but only adds a couple of lines of overhead when there are no `Listeners`.

Performance is impacted far more by having many `Listeners` attached to each `ObservableValue`, especially if those `Listeners` are complicated or trigger other `Listeners`.  But, of course, even if you have `Properties` with thousands of `Listeners` added, if your implementation is such that no code ever calls `set()` on those `Properties`, then those `Listeners` will never have an impact on performance.

My personal experience from looking into poorly performing JavaFX applications is that most (if not all) issues are caused by incorrect layout practices, and "self-inflicted" problems caused by a lack of understand about how JavaFX works.  Doing things like creating `Nodes` on the fly, or rebuilding `TableCell` layouts in response to `TableCell.updateItem()` are far more likely to have an impact on performance that having large amounts of `Observable` data.

This is especially a issue for beginners, and it's easy to see how they can get an impression that JavaFX is "heavyweight" and that performance issues need to "worked around".  

There's probably a reason why JavaFX doesn't contain any equivalent to `ImmutableStringValue`, and that reason probably is that it offers virtually no performance benefit to returning `SimpleStringProperty` as an `ObservableValue`.  

# Impact on On-Line Resources

I wasted an afternoon searching around for an tutorials or on-line resources that talked about `ReadOnlyObjectWrapper` and its use in the example code for `TableColumn`.  

Sadly, as is the case with most JavaFX topics, there isn't really a lot out there.  But I did notice a few things:

## PropertyValueFactory is Still Mentioned

This is a little off topic, but somewhat relevant...

The JavaDocs aren't clear that `PropertyValueFactory` was a stop-gap solution until something better could be implemented.  That something better was `Lambda` functions and no one should be using `PropertyValueFactory` any more.  There's no advantage to using `PropertyValueFactory` over using a `lambda` function like this:

``` kotlin
  column.CellValueFactory = Callback { it.value.someProperty() }
```
Sadly, you can still see it being recommended...

This article on [DelftStack](https://www.delftstack.com/howto/java/javafx-setcellvaluefactory/) was written in 2024 doesn't even mention any other way to do this.  It's a fairly long example that boils down to repeated versions of this:

``` java
TableColumn FirstNameCol = new TableColumn("First Name"); // Create a column named "First Name"
FirstNameCol.setMinWidth(100); // Set the minimum column width to 100
FirstNameCol.setCellValueFactory(
    new PropertyValueFactory("firstName")); // Populate all the column data for "First Name"
```

This page on [Jenkov.com](https://jenkov.com/tutorials/javafx/tableview.html#tablecolumn-cell-value-factory) from 2021 does the same thing.

This page on [CoderScratchpad.com](https://coderscratchpad.com/javafx-treetableview-building-hierarchical-data-displays/) from 2023 deals with `TreeTableView` and it uses the equivalent `TreeItemPropertyValueFactory`.


## StackOverflow Does a Better Job



## Usages in GitHub Projects


## Not Mentioned At All

This isn't really a bad thing in and of itself.  The best practice is absolutely to compose your `TableView` models from `ObservableValues` of whatever types and simply return references to them in the `CellValueFactory Callback`.  

This is what you see in 90% of the on-line tutorials that I could find.  Which is good, as far as it goes, but it doesn't really add anything to what you could learn by looking at the JavaDocs themselves.  And where's the value in that?

## Just Plain Bad Advice

Also, a bit off topic but I think it's worth mentioning as a commentary on the quality of tutorials out there on the web.

Take a look at this page on [Demo2s.com](https://www.demo2s.com/g/java/how-to-customize-the-functionality-of-setcellvaluefactory-in-javafx-s-tableview-in-java.html) with the title "How to customize the functionality of setCellValueFactory in JavaFX's TableView in Java".

It's your staple example of a "Person" class with a name and something else, in this case an "age", both implemented as `Properties`.  We have the usual boilerplate example for a `TableView` with a "Name" and an "Age" column.  Then it tackles the case where you want to combine the two elements together into a single column:

``` java
TableColumn<Person, String> customColumn = new TableColumn<>("Custom Column");
customColumn.setCellValueFactory(cellData -> {
    Person person = cellData.getValue();
    // Customize the display value based on your requirements
    String customValue = person.getName() + " (Age: " + person.getAge() + ")";
    return new SimpleStringProperty(customValue);
});
```
At least here, it doesn't wrap the calculated value inside `ReadOnlyObjectWrapper`, and uses `SimpleStringProperty` instead.  

The rest of the implementation is rubbish, though.  

He's taken perfectly good `Observables`, yanked their current values from them, combined those values together with some text to create a new `String` and then stuffed the results back into a `SimpleStringProperty` for compatibility.  Now, if either of those two `Observables` changes its value, the `TableView` will **NOT** be updated.

Absolutely, the correct approach in this situation is to create a `Binding`.  Probably `Bindings.concat` would be best:

```
  return Bindings.concat(person.nameProperty(), " Age: ", person.ageProperty(), ")");
```
This will always reflect the current values in `Person`.
