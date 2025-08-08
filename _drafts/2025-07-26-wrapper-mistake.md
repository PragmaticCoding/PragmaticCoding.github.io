---
title:  "Rant: Misleading JavaDocs"
date:   2025-07-26 12:00:00 -0500
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

JavaDocTC: https://openjfx.io/javadoc/23/javafx.controls/javafx/scene/control/TableColumn.html
JavaDocFXC: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/FXCollections.html

excerpt: "How ReadOnlyObjectWrapper is misused in the JavaDocs introductory section for TableColumn, and how this has propagated throughout various on-line tutorals and resources."
---

# Introduction

I know that I've complained in the past about the quality of some of the official JavaFX JavaDocs, especially the quality of the introductory text for many pages, and of the examples that are given.  Usually, however, that text is *technically* correct but has some subtle nuances that are critical but not obvious to beginners - the people most likely to be reading that section.  

This is a problem.  A pretty big problem, actually.  

But, once in a while, I come across something that is truly misleading, if not actually wrong.  And in those cases, what I usually find is that other sources on the web just repeat it without question.

And that's an even bigger problem.  

So let's look at one example, and see how the JavaDocs is misleading, and how nobody writes about it anywhere else...

# The Issue

If you look at the introductory section in the JavaDocs for [TableColumn]({{page.JavaDocTC}}), you'll see that it discusses methods to populate data into a `TableColumn` via `TableColumn.setCellValueFactory()`.  In the first example, it talks about the normal case where the field is already defined in the `TableView Item` class as an `ObservableValue` of some type.  

So far, so good.

Then it goes on to deal with the situation where the field is not already an `ObservableValue`, but just a primitive or regular Java object, like a `String`.  Let's take a look at what it says:

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

I'll repeat what it says in the text, "...it is possible to wrap the returned value in a ReadOnlyObjectWrapper instance...", because this emphasizes that this isn't some typo or mistake in the attached code sample.  

And, if you are like me, when you read this you get the impression that `ReadOnlyObjectWrapper` is some kind of utility class that allows you to turn a non-`Observable` data element into an `Observable` for the sake of compatibility.  The name itself seems to imply this, it's a "read-only object wrapper"!

In other words, any time you need to pass an `Observable`, but all you have is a primitive, stuff it into `ReadOnlyObjectWrapper` and off you go.  We are lead to believe - from this example - that this is the use case that `ReadOnlyObjectWrapper` was designed for.

But is it?

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

But there is a loophole.  There is no way to instantiate an `ObservableIntegerValue` because it is *just* an interface.  The only concrete classes that you can actually instantiate out of the entire class/interface structure is `SimpleIntegerProperty` and its subclass `ReadOnlyIntegerWrapper`.  You can also extend `IntegerBinding` and create a custom subclass that you could instantiate, and that would also implement `ObservableIntegerValue` - but we're not talking about that here.

Now, what this means is that any `ObservableIntegerValue` created via the method in the code snippet above is going to actually be instantiated as `SimpleIntegerProperty`.  This makes sense when you think about it.  What good is an `Observable` that can truly never change?  No `Listeners` could ever fire and it would be no better than any generic wrapper class.  

But this means that any `ObservableIntegerValue` can be cast to at least `IntegerProperty` by client code.  And, of course, `IntegerProperty` supports `IntegerProperty.set()`.  Like this:

``` kotlin
val x: ObservableIntegerValue = SimpleIntegerProperty(0)
println("Is SimpleIntegerProperty: ${x is SimpleIntegerProperty}")
(x as IntegerProperty).set(5)
```
This code will run, will print out "Is SimpleIntegerProperty: true", and will change the value to 5 without error.

But what if you have some implementation where it is really, really, really important that `IntegerProperty.set()` is never available to client code?

Enter `ReadOnlyIntegerProperty` and `ReadOnlyIntegerWrapper`.  These two classes together solve the enigma of having a truly read-only `Observable` that, at the same time, might have its value change and fire `Listeners`.

First off, `ReadOnlyIntegerWrapper` extends the concrete class `SimpleIntegerProperty` and adds a single method called `getReadOnlyProperty()`.  If you look at the JavaDocs for `ReadOnlyIntegerWrapper` you'll see this:

> This class provides a convenient class to define read-only properties. It creates two properties that are synchronized. One property is read-only and can be passed to external users. The other property is read- and writable and should be used internally only.

It says that the two properties are "synchronized", but they aren't really.  The `get()` method of the `ReadOnlyIntegerProperty` that is returned from `getReadOnlyProperty()` actually delegates to the originating `ReadOnlyIntegerWrapper`.  In fact, the `ReadOnlyIntegerProperty` doesn't actually have a `value` at all!  You can add `Listeners` to a `ReadOnlyIntegerProperty`, but they are actually fired by the originating `ReadOnlyIntegerWrapper` when its `value` is invalidated.  

Finally, `ReadOnlyIntegerProperty` isn't implemented by any classes that have a `value` or that support `set()`.  This means that there is no way to cast a `ReadOnlyIntegerProperty` to any class that does support `set()`.  It is truly read-only, and yet will still behave like an `Observable`.  The linkage back to the `ReadOnlyObjectWrapper` is only available to the originating code that already has that `ReadOnlyObjectWrapper`.

The `ReadOnlyIntegerWrapper`, however, can be treated just like an `IntegerProperty` or `SimpleObjectProperty`.  In fact, you could, if you wanted, use `ReadOnlyIntegerWrapper` instead of `SimpleIntegerProperty` everywhere in your applications and it wouldn't make any difference to how they work.

As far as that goes, the code example in the `TableColumn` JavaDocs is fine.  It's the context around it that causes issues...

# Back to the JavaDocs for TableColumn

If you never call `ReadOnlyIntegerWrapper.getReadOnlyProperty()`, then you might as well just use `SimpleIntegerProperty`.

Let's look at the example code from the JavaDocs again:

``` java
 firstNameCol.setCellValueFactory(new Callback<CellDataFeatures<Person, String>, ObservableValue<String>>() {
     public ObservableValue<String> call(CellDataFeatures<Person, String> p) {
         return new ReadOnlyObjectWrapper(p.getValue().getFirstName());
     }
  });
```
You can see that they never call `ReadOnlyObjectWrapper.getReadOnlyProperty()`.  And that value is returned as an `ObservableValue<String>`, so `getReadOnlyProperty()` isn't going to be available to whatever calls this `CellValueFactory` (unless it casts it to `ReadOnlyObjectWrapper`).  

This means that they might as well have just returned `SimpleObjectProperty` here instead.  But they didn't.

Let's not forget that this `Callback` is only ever going to be invoked by some internal code in `TableView` or `TableColumn`.  I think that we can safely assume that whatever code that is, it's not going to cast the results to `ObjectProperty` and then call `set()` because it's part of the JavaFX library itsef.  So there is no value at all to going through the process of creating `ReadOnlyObjectWrapper` in order to return a `ReadOnlyObjectProperty`.

However, if we go back to that explanatory text,

> ...it is possible to wrap the returned value in a ReadOnlyObjectWrapper...

we see that this is a deliberate choice. What were they thinking?

This is the problem.  And it's a big problem.

I strongly suspect that most programmers that read this explanation (and probably the person that wrote it), view `ReadOnlyObjectWrapper` as something that works like this:

``` java
public class StringConstant extends StringExpression {

    private final String value;

    private StringConstant(String value) {
        this.value = value;
    }

    public static StringConstant valueOf(String value) {
        return new StringConstant(value);
    }

    @Override
    public String get() {
        return value;
    }

    @Override
    public String getValue() {
        return value;
    }

    @Override
    public void addListener(InvalidationListener observer) {
        // no-op
    }

    @Override
    public void addListener(ChangeListener<? super String> observer) {
        // no-op
    }

    @Override
    public void removeListener(InvalidationListener observer) {
        // no-op
    }

    @Override
    public void removeListener(ChangeListener<? super String> observer) {
        // no-op
    }
}
```
This is clearly a "wrapper" around an immutable value, and this wrapper behaves *exactly* as you would expect an `ObservableValue<String>` to behave.  There's no point in keeping track of the `Listeners` that are added, or to actually try to remove them because they can **never** fire.  

This class does what appears to be needed - and certainly what is implied by the `TableColumn` JavaDocs.  Our source data isn't actually `Observable`, so there's no value in really implementing `Observable` methods.  However, it does provide type compatibility with `ObservableValue` and there is zero possibility that using this would ever "break" any client code that received it when expecting a "real" `ObservableValue`.

This is how I pictured `ReadOnlyObjectWrapper` working when I first read the `TableColumn` JavaDocs.  I'll bet you did too.  And just about everyone else.

I know that I read the JavaDocs for `TableColumn` long, long before I ever looked into `ReadOnlyObjectWrapper`.  I'd be willing to bet that most programmers had exactly the same experience.  Everybody gets to `TableView`, and therefore, `TableColumn` pretty early on when you start to work with JavaFX.  But you've probably never read the JavaDocs for `ReadOnlyObjectWrapper`.

You may be wondering why `StringConstant` is shown in Java, and not Kotlin?  That's because this is an actual class from inside the JavaFX library.  Unfortunately, it's in `com.sun.javafx.binding` which is an internal library, and you shouldn't use it in your applications.  There's a complete suite of these classes, for all of the various specialty types of `ObservableValues`.

Internally, the JavaFX library uses `StringConstant` exactly the way that you'd expect.  This is from the `Bindings` library:

``` java
public static BooleanBinding equal(final ObservableStringValue op1, String op2) {
    return equal(op1, StringConstant.valueOf(op2), op1);
}
```
The method that really does the work is:

``` java
private static BooleanBinding Bindings.equal(final ObservableStringValue op1,
                                             final ObservableStringValue op2,
                                             final Observable... dependencies)
```

It requires that both `op1` and `op2` are `ObservableStringValues`.  So in `Bindings.equal(final ObservableStringValue op1, String op2)` the `String` operand is stuffed into `StringConstant` simply to make it compatible with `ObservableStringValue`.

## Could it Be About Performance?

I think that a lot of programmers obssess about a percieved "overhead" when using `Observables`.  This is especially true when they have large amounts of data, and worry about excessive memory usage and performance issues when processing large amounts of `Obseravble` data.  

My general sense from looking at the internal code to see how these classes actually work, is that they are designed to minimize memory and performance overhead, and that an `Observable` without any `Listeners` added won't even have data storage allocated for `Listeners` until the first one is added.  There is code that checks for `Listeners` whenever an `ObservableValue` is changed, but only adds a couple of lines of overhead when there are no `Listeners`.

Performance is impacted far more by having many `Listeners` attached to each `ObservableValue`, especially if those `Listeners` are complicated or trigger other `Listeners`.  But, of course, even if you have `Properties` with thousands of `Listeners` added, if your implementation is such that no code ever calls `set()` on those `Properties`, then those `Listeners` will never fire, and never have an impact on performance.

My personal experience from looking into poorly performing JavaFX applications is that most (if not all) issues are caused by incorrect layout practices, and "self-inflicted" problems caused by a lack of understanding about how JavaFX works.  Doing things like creating `Nodes` on the fly, or rebuilding `TableCell` layouts in response to `TableCell.updateItem()` are far more likely to have an impact on performance that having large amounts of `Observable` data.

This is especially a issue for beginners, and it's easy to see how they can get an impression that JavaFX is "heavyweight" and that performance issues need to be "worked around".  It's very possible that this is why `ReadOnlyObjectWrapper` is assumed to work the same was as the internal `ObjectConstant` actually does.

I strongly suspect that the reason that `ObjectConstant` isn't exposed in the public API for JavaFX is that, in real world applications, there's just no performance advantage to using it over `ObjectProperty`.

# Other On-Line Resources

I wasted an afternoon searching around for tutorials or on-line resources that talked about `ReadOnlyObjectWrapper` and its use in the example code for `TableColumn`.  

As is the case with most JavaFX topics, there isn't really a lot out there.  But I did notice a few things:

## PropertyValueFactory is Still Mentioned

This is a little off topic, but somewhat relevant...

The JavaDocs aren't clear that `PropertyValueFactory` was a stop-gap solution until something better could be implemented.  That something better was `Lambda` functions and no one should be using `PropertyValueFactory` any more.  There's no advantage to using `PropertyValueFactory` over using a `lambda` function like this:

``` kotlin
  column.CellValueFactory = Callback { it.value.someProperty() }
```
At the same time `PropertyValueFactory` uses reflection and will generate run-time errors if the reflection fails.  It will not, however, generate compile-time errors if the `Property Bean` hasn't been implemented properly, or if the name of the property is mispelled in the constructor.  None of these problems exist with the lambda approach.

Sadly, you can still see `PropertyValueFactory` being recommended...

This article on [DelftStack](https://www.delftstack.com/howto/java/javafx-setcellvaluefactory/) was written in 2024 doesn't even mention any other way to do this.  It's a fairly long example that boils down to repeated versions of this:

``` java
TableColumn FirstNameCol = new TableColumn("First Name"); // Create a column named "First Name"
FirstNameCol.setMinWidth(100); // Set the minimum column width to 100
FirstNameCol.setCellValueFactory(
    new PropertyValueFactory("firstName")); // Populate all the column data for "First Name"
```

This page on [Jenkov.com](https://jenkov.com/tutorials/javafx/tableview.html#tablecolumn-cell-value-factory) from 2021 does the same thing.

This page on [CoderScratchpad.com](https://coderscratchpad.com/javafx-treetableview-building-hierarchical-data-displays/) from 2023 deals with `TreeTableView` and it uses the equivalent `TreeItemPropertyValueFactory`.

## Just Plain Wrong!

This [SolutionFall.com](https://solutionfall.com/question/what-is-the-purpose-of-using-readonlyobjectwrapper-in-tablecolumns-setcellvaluefactory-method/) site was a gem!  This page is in a question & answer format.  The question explains the use of `ReadOnlyObjectWrapper` in the JavaDocs and then asks this:

>  This raises the question of the purpose and necessity of using `ReadOnlyObjectWrapper` in this context. The practical application of `getReadOnlyProperty()` in comparison to simply casting a `SimpleObjectProperty` to an `ObservableValue` is unclear. This discrepancy prompts confusion and leaves the question of whether this is an error in the JavaDocs or if there is a crucial aspect missing from the understanding of these classes.

That summarizes the problem in a nutshell.  The "answer", however, doesn't seem to get it...

> The purpose of using `ReadOnlyObjectWrapper` in `TableColumn`’s `setCellValueFactory()` method is to provide a way to observe changes in the value of a property. It allows you to create a binding between the value of a property and the cell in the table column so that any changes in the property value are automatically reflected in the table column.<br><br>
> By using `ReadOnlyObjectWrapper`, you can create a read-only wrapper around the property value, making it observable. This is particularly useful when working with JavaFX TableView components where you want the table to automatically update when the underlying data changes.<br><br>
> In summary, `ReadOnlyObjectWrapper` provides a convenient way to make a property value observable and enable two-way binding with the table column in JavaFX applications.

I'm not sure what the "two-way binding" rubbish at the end is about since a read-only `Property` shouldn't be bidirectionally bindable.  

Now, the part about, "By using `ReadOnlyObjectWrapper`, you can create a read-only wrapper around the property value, making it observable.", is exactly the misunderstanding that we've been talking about here.

Unfortunately, this is about the quality of information you can expect on the web.

## ChatGPT

I took the question from SolutionFall.com and plugged it into ChatGPT, just to see what it would say.  I was a little surprised because it gave a mostly correct answer.  It certainly understood the question!  

It output a lot of stuff, but ended with this:

> ## Summary: Key Takeaways
> + Not exactly an error, but potentially misleading or idealized documentation.
> + The JavaDoc describes the intended pattern for exposing read-only properties.
> + The actual implementation might skip wrapping or exposing read-only views if:
>   + The mutability isn't a practical concern.
>   + The performance cost of creating a wrapper is deemed unnecessary.
>   + The API already ensures users can't access the raw property object.
> + The use in TableColumn probably reflects a pragmatic choice, not necessarily a design inconsistency — but it does blur the ideal boundary of read-only exposure.
+ It's not necessarily a JavaDocs bug, but it may give a misleading impression of how rigorously getReadOnlyProperty() is used in practice.

Which is not too bad, really.  The last point is pretty much dead on.

## StackOverflow Does a Better Job

The JavaFX community of programmers that answer questions on StackOverflow.com is dominated by about 4 or 5 members.  If you see answers from "James_D", "jewelsea", "SedJ601" or "slaw", then they are generally going to be high quality.  

In virtually every question that they comment on or answer (and that's virtually all of them), they recommend to not use `PropertyValueFactory`, and give examples using `Simple{type}Property` instead of `ReadOnly{type}Wrapper`.

## Not Mentioned At All

This isn't really a bad thing in and of itself.  The best practice is absolutely to compose your `TableView` models from `ObservableValues` of whatever types and simply return references to them in the `CellValueFactory Callback`.  In that case you don't need to worry about putting your immutable value into an `ObservableWrapper` at all.

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
This will always reflect the current values in `Person`.  It's a lot simpler, too.

# Conclusion

Sure, this is a rant, but I don't think it's an unreasonable rant.

The JavaDocs should be the single source of reliable information about the entire library.  And the introductory sections should be extremely accurate, "best practices" for all beginners.  They should be clear and free of subtleties that are going to be missed by non-experts.

But, all too often, that's not what we get.  In this particular case we get something that seems almost deliberately misleading.

It's not like `ReadOnlyObjectWrapper` is an obscure, little-used, element of the library.  Many, many JavaFX objects return `ReadOnlyObjectProperty`, and **all** of those implemented internally via `ReadOnlyObjectWrapper`.  If you call `Region.widthProperty()`, for instance, you'll get a `ReadOnlyDoubleProperty`.  If you check the source code, you'll find this:

``` java
public final ReadOnlyDoubleProperty widthProperty() {
    if (this.width == null) {
        this.width = new ReadOnlyDoubleWrapper(this._width) {
            protected void invalidated() {
                Region.this.widthChanged(this.get());
            }

            public Object getBean() {
                return Region.this;
            }

            public String getName() {
                return "width";
            }
        };
    }

    return this.width.getReadOnlyProperty();
}
```
This pattern is repeated over and over, all through the library.  

So, as far as I can see, it's not unreasonable to expect the JavaDocs to handle this properly, and to understand how `ReadOnlyObjectWrapper` is supposed to be used.

Of course, the rest of the web treats the JavaDocs like they are that "single source of reliable information", and almost every tutorial that you are likely to run across simply parrots what the JavaDocs say.  
