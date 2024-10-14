---
layout: single
title: Understanding the Kotlin Examples
short_title: Kotlin Examples
toc: true
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
permalink: /kotlin/kotlin-examples
skip_link: false
intro: /kotlin/intro
---

# Why Kotlin?

Throughout many articles on this website, especially the later ones, you'll find that the example code is written in Kotlin.  There's a very good chance that you've never seen Kotlin before, and an even lesser chance that you've actually written any programs in Kotlin.

So, why use Kotlin in the example code?

Kotlin is very much like Java but way less clumsy.  There are features in Kotlin that fix issues with Java that you didn't realize were issues with Java until you see them fixed in Kotlin.  

One of the things about Kotlin that makes it attractive for use in my articles is that it clears away a lot of the bloat that comes from JavaFX boilerplate.  This lets me write very simple code that's pared down to bare minimum needed to demonstrate what I'm talking about.  I think this makes it more effective for example code snippets in the articles.

And also, the JavaFX articles on this website are about JavaFX, not Java.  The JavaFX concepts don't change when you use them in Kotlin, and all of the ideas are equally usable in both Kotlin and Java.  Some of these articles take a large amount of experimental and investigative coding, and most of the examples are copied out of working programs that I wrote when preparing the articles.  I do that coding in Kotlin because Kotlin is just easier for me to use, and I'm not going to re-write it in Java for the articles and have less effective code examples.

Finally, JavaFX is not, unfortunately, a good thing for a beginning programmer to take on.  It demands that you have a very good handle on multi-threading, classes, interfaces and inheritance, and that you understand advanced programming concepts in Java.  If your first Java program was "Hello World" last week, then you probably shouldn't be messing around with JavaFX.  

Any programmer proficient enough in Java to tackle JavaFX should have no problem understanding Kotlin code.  The truth is, when you look at Kotlin code as an experienced Java programmer, your first guess about what that code is doing is probably pretty close to correct.  You don't need to understand the syntax of Kotlin to the point where you can write Kotlin code to be able to read it and understand most of it.

Finally, finally...Maybe, just maybe you'll look at these articles and think to yourself, "That Kotlin stuff is really neat!  I should try it out for myself".  That would be awesome.

# Elements of Kotlin

This section outlines the core elements of Kotlin so that you can understand them when you encounter them in the articles.  I'm going to try to present these elements in an order that builds on the parts that have gone before, so it's best just to read it that way...

## Variable and Field Declarations

You'll often see lines of code that look like this:

``` kotlin
val someVariable : StringProperty = SimpleStringProperty("abc")
```
This initializes `someVariable` as the type `StringProperty` by calling the constructor of `SimpleStringProperty`.  The `val` argument specifies that this is an immutable reference, equivalent to `final` in Java.  

Kotlin does not terminate lines with ";", nor does it use `new` to invoke constructors.

The Java equivalent is  this:

``` java
final StringProerty someVariable = new SimpleStringProperty("abc");
```
If a variable or field is not to be immutable, then `var` is used in place of `val`.  In Kotlin every variable and field has to be declared as `val` or `var` and `val` is prefered whenever possible.  I actually feel a little dirty when I declare a variable as `var` nowadays.

Finally, you can skip the type declaration if the compiler can infer it from the initialization:

``` kotlin
var counter = 12
```
Will result in `counter` being typed as `Int`.

## Property Access Notation

First off, Kotlin calls class fields, "Properties", which can be a little confusing when talking about code in a JavaFX context where there are also JavaFX `Properties`.  In article text, you'll always see JavaFX `Properties` notated as code elements while Kotlin properties won't have any special notation.  More often, the articles will just use "field" instead to avoid confusion.  

Kotlin's properties aren't strictly speaking the same as Java fields, and have extra features including internal getters and setters.  One of the benefits of this is "Property Access Notation", where you just refer to the property directly and Kotlin will automatically invoke the associated getter or setter.  This feature is backwards compatible with Java, and if Kotlin detects getters and setters in a Java class, then you can use Property Access Notation, and Kotlin will call the getter or setter for you.  Like this:

``` kotlin
val label = Label()
label.text = "abc"
```
Which is equivalent to:
``` java
Label label = new Label();
label.setText("abc");
```
This works even though `Label.setText()` delegates to `Label.textProperty().set()`.

## Lambda Expressions and Lambdas as Function Parameters

In Kotlin lambda expressions are always encased in "{}", including the parameter declaration, even if they are single line.   Like this:

``` kotlin
val someFunction = { p1, p2 -> doSomething(p1, p2) }
```
If there is only a single parameter, it may be omitted and refered to in the function as "it":

``` kotlin
val someFunction = { doSomething(it) }
```
In Kotlin, you'll see lambdas passed as function parameters quite often.  But not like this:

```
callFunction(a, b, { p1, p2 -> doSomething(p1, p2) })
```
because when the last parameter in a function call is a function, it can be moved outside of the "()", like this:

``` kotlin
callFunction(a, b) { p1, p2 -> doSomething(p1, p2) }
```
Which looks much cleaner, especially when the lambda is multi-line.

If the only parameter in a function call is a function, then the "()" may be omitted entirely.  

``` kotlin
`callFunction { p1, p2 -> doSomething(p1, p2) }`
```
If you are writing a lambda as an implementation of a functional interface you can just put it after the name of the functional interface.  For instance:

``` kotlin
someProperty.addListenter(ChangeListener {obVal, oldVal, newVal -> doSomething(oldVal, newVal) })
```
And finally, if you aren't going to use one or more of the parameters in your lambda, then you can replace them with "_":

``` kotlin
someProperty.addListenter(ChangeListener {_, oldVal, _ -> doSomething(oldVal) })
```

## Operator Overloading

Kotlin supports a set of functions which can be implemented on any class as operators.  The most common example of this on this website is the use of `List.plusAssign()` which uses the "+=" operator.  This is the same as `List.add()`.  For instance:

``` kotlin
hBox.children += Label("ABC")
```
which is the equivalent to:

``` java
hBox.getChildren().add(new Label("ABC"));
```

## Scope Functions

Kotlin has a concept called "Scope Functions" that is very handy with JavaFX.  Mostly commonly, in the articles on this website you'll see `apply{}` used:

``` kotlin
val label = Label().apply{
   text = "abc"
   styleClass += "label-style"
}
```
This is the equivalent to:

``` java
val label = new Label();
label.setText("abc");
label.getStyleClass().add("label-style");
```
The `apply{}` function can be called on **any** class, and will pass the object that it is called on to the lambda as its only parameter and will be named, `this`.  Just like in Java, if `this` can be inferred, then it can be omitted.  The `apply{}` function returns the object that it was called on.

The big advantage of this in JavaFX is that you can configure `Nodes` and add them to a layout without having to declare them as variables, like this:

``` kotlin
hbox.children += Label().apply {
  text = "abc"
  styleClass += "label-style"
}
```

## Function Declarations

Kotlin function declarations use a format similar to variable and field declarations:

``` kotlin
private fun someFunction(p1 : String, p2: Int) : String {
    .
    .
    .
  return someString  
}
```

Alternatively, when the return type can be inferred:

``` kotlin
fun formatName(fName : String, lName : String ) = lname + ", " + fName
```
This is useful with Scope Functions:
``` kotlin
private fun createContent() : Region = BorderPane().apply {
  center = createCentre()
  bottom = createBottom()
  padding = Insets(20.0)
}
```

## Null Safety
One of the big advantages to Kotlin is its integrated `Null` safety.  It's virtually impossible to get an NPE in Kotlin unless you do some really silly things - and you won't find those on this website.

If a Kotlin variable can be `Null` then it must be declared as such by appending a "?" to its type:

``` kotlin
var someValue : Int? = 54
```
Note, however, that `Int?` is not the same as `Int`.  You cannot do this, for example:

``` kotlin
var someValue : Int? = 54
var otherValue = someValue + 5
```
Because `someValue` is not an `Int`, it's a `Nullable Int`.  In a way, this is very much comparable to Java's `Optional`.  

In order to access the value inside a nullable type, you must specify how to deal with the `Null` case.  The most straight-forward is to use the "Elvis Operator":

``` kotlin
var someValue : Int? = 5
var otherValue = 2 + someValue ?: 0
```
In this example, `otherValue` will be set to `2 + someValue` unless `someValue` is `Null` in which case it will be set to `2 + 0`.

If a nullable class has methods or fields, you can use the null safety operator "?" to call them:

``` kotlin
var label : Label?
  .
  .
  .
val textLength = label?.text?.length ?: 0
```
Here, `label?.text` will return `Null` if `label` is `Null`, otherwise the `text` value will be returned.  Similarly, `length` wil be returned only if `label?.text` is non-null.  

You can also invert the logic and use the `let{}` scope function to perform an action if the value is not `Null`:

``` kotlin
var label : Label?
  .
  .
  .
var textLength = 0
label?.let{ textLength = it.text.length }
```
Unlike `apply{}`, `let{}` passes the object it is called on as `it`.

## If Statements

Kotlin does not have a ternary operator, but the `if` statement is also an expression and returns a value.  So you can do this:

``` kotlin
val someValue = if (x > 7) then y else z
```

## Class Definitions, Constructors, and init{}

If Kotlin classes require values passed in constructors which are simply assigned to fields, then you can do this:

``` kotlin
class SomeClass(private val name : String, private var age : Int) : SomeSuperClass(), SomeInterface, SomeOtherInterface {}
```
Here the fields `name` and `age` are defined, and calls to the constructor will need to supply values for it.  `SomeClass` extends `SomeSuperClass` and implements two interfaces.

If there is additional code that needs to be run when the class is instantiated, it can be put into an `init{}` block.  This runs just like the code in a Java constructor:

``` kotlin
class SomeClass(name: String, age : Int) {
  private nameProperty : StringProperty = SimpleStringProperty(name)
  private ageProperty : IntegerProperty = SimpleIntegerProperty()

  init {
     ageProperty.value = age
  }
}
```
In this example, both `name` and `age` are simply passed to the constructor as values, and do not define fields (note the lack of `var` and `val` in their definitions).  The way that both `nameProperty` and `ageProperty` are set from these parameters are roughly equivalent, but the method for `nameProperty` is more concise, and how you'll see it on this website.

# Learning More
This article is by no means intended to be a comprehensive list of Kotlin syntax, but it will give you enough information to understand most of the code snippets that you'll find on this website.

If you want to know more, (and you should), the official Kotlin documentation can be found [here](https://kotlinlang.org/docs/home.html).
