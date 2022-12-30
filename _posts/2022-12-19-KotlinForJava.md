---
title:  "Kotlin For Java Programmers"
date:   2022-12-19 12:00:00 -0500
categories: kotlin
logo: /assets/logos/JavaFXLogo.png
permalink: /kotlin/kotlin_for_java_programmers
excerpt: Need to understand a Kotlin program, but you only know Java?  This article should give you everything you need to know to get started.
---

# Introduction

The first time I ever looked at a Kotlin program was when one of my developers found a cool JavaFX library called "DirtyFX".  We wanted to understand how some of it worked, and when we looked at the source code on GitHub we were stymied when we saw it was in Kotlin.  

It seemed impenetrable.

Years later, I've gone back to Kotlin and learned it.  It's awesome and, if you're a competent Java programmer, you can learn enough to start using it with a 4 hour YouTube tutorial.  Then it takes about a week or so to get comfortable, and somewhat longer to get proficient at it.  

This article isn't designed to replace a 4 or 5 your YouTube tutorial, it's intended to give a typical Java programmer enough information to understand a typical Kotlin program in a much shorted period of time.  

One more thing.  If you study Kotlin for any length of time, you'll come across the term "idiomatic Kotlin".  This is the idea that while Kotlin shares the JVM, just writing code the way you would in Java but in Kotlin isn't going give you something that *feels* like Kotlin.  There's definitely a Kotlin approach to coding, and coding in that way gives you idiomatic Kotlin.  In order to *write* code this way, you do really have to have a solid grasp of a lot of Kotlin, but once you get used to some of the quirky things you'll see in Kotlin code you should be able to easily understand it.

## Why Kotlin

When I learned Kotlin I realized that there was lots of stuff in Java that bugged me (or should have), that Kotlin just fixes in a seamless and natural way.  Null safety is a clear example of that.  Sure, in Java you can use `Optional`, but Kotlin's approach is nicely integrated into the language in such a way that Null safety is virtually mandatory in any code you write.

In general, Kotlin code is shorter and more intuitive than the equivalent in Java, and is therefore easier to read and understand.  

# Class Structure

Let's start at the top, class structure.  We'll use this sample code to talk about a fet topics:

``` kotlin
class MyClass(val property1 : Int, private var property2 : Int = 0) : SomeInterface {

    var property3: SomeInterface = SomeImplementation()
    protected val property4 = mutableListOf("abc", "def")
    private var property5: Int

    init {
      property5 = property1 + 25
    }


    fun someMethod(param1 : String, param2 : Double = 3.2) : Boolean {
      val localVariable : Int = 44
          .
          .
          .
       return booleanValue
    }
}
```

## Declaring Types

The first thing that you'll notice is that there aren't any semi-colons at the ends of the lines.  You can use them, but you don't need to.

The second thing that you'll notice is that type declarations are of this structure:

``` kotlin
keyword name : type
```
This structure is the same whether you're declaring a field, a variable, a class or a method.  

If you are initializing a variable and it can be reasonably inferred as to what it is, then you do not need to explicitly specify the type:

``` kotlin
var abc = 5
```
Will give you Int.  However, if you want Long, then you need to specify it:

``` kotlin
var abc : Long = 5
```

## Instance Variables

I'm very specifically using the term "instance variables" because Kotlin has a different structure for static elements, and these are NOT fields.  Kotlin calls them "Properties".

A Kotlin property is actually a structure that is backed by a "field".  There is a default getter and setter for the property, and should you chose to override them you can access the backing field inside your getter/setter code.  For now, this is all you need to know about this.

One advantage to Kotlin is that it automatically invokes the getter or setter for a property when it is directly accessed via dot notation in your code.  Let's look at this snippet:

``` kotlin
myClass.property3 = myClass.property4[2]
```
This is completely valid code.  It calls the setter for `property3` and the getter for property4.  On top of that, it calls `List.get()` by using `[]` construct.  You can also do this:

``` kotlin
myClass.property4 += "xyz"
```
Here it accesses the getter for `property4` and then calls `List.add()` via the `+=` operator.  This is called ["Operator Overloading"](https://kotlinlang.org/docs/operator-overloading.html).  You can define your own overloaded operators by defining methods on a class with particular names.  

If this alone doesn't sell you on Kotlin, I don't know what will.  You get all of the benefits of getters and setters with none of the overhead!  And the operator overrides make your code look the way you always wanted it to in Java, but couldn't be.

## Mutability

By now you should have noticed the `val` and `var` prefixes for property and variable declarations.  These indicate whether or not the reference is mutable.  It's not optional in Kotlin to specify this.  Functionally, `val` is very close to the `final` keyword in Java, with the exception that there's no default to "not final".  

The convention is to declare everything as immutable using `val` unless you specifically have a need for it to be mutable.  In practice, this makes a huge difference in the readability of code since you don't need to chase through the code looking for any places that any variable might have been changed.  This is also a boon when writing multi-threaded code.

Personally, I find that 95%+ of my variables are declared with `val`.  I almost feel guilty when I use `var`.

Lists are generally considered immutable unless you declare them as mutable.  Remember that declaring a `List` variable as mutable is not the same as a mutable `List`.  

Consider this:

``` kotlin
val list1 = listOf("a", "b")
var list2 = listOf("d", "e")
val list3 = mutableListOf("x", "y", "z")
```
The only operations you can perform on `list1` are `List.get()` type operations.  You cannot add, remove or change any of the elements.  The same restrictions apply to `list2`, however you can replace it with a new immutable list, meaning that:

``` kotlin
list2 = listOf("dog", "cat", "budgie")
```
would be allowed.  You can add or remove elements to `list3`, but but you cannot instantiate a new List as `list3`.

## Methods

Methods in Kotlin are called [Functions](https://kotlinlang.org/docs/functions.html) and declared using the `fun` keyword.  The return type of a function is declared the same way as for a variable and the body of the method is enclosed in `{}` if there is more than one line.  If there is more than one line, then you can just use `=` and put the code.  For instance:

``` kotlin
fun doubleIt(x: Int): Int {
  return x * 2
}

fun double(x: Int): Int = x *2
```
are equivalent. In the second case, the type declaration can be left out as the compiler can infer it.

`Void` functions either return `Unit` or just leave out the return type altogether.  

Functions can have any number of parameters declared and those parameters can have default values.  Consider this:

``` kotlin
fun multiplyIt(x: Int, multiplier: Int = 2) = x * multiplier
```
Which can be called in the following ways:

``` kotlin
val answer = multiplyIt(3,2)
val answer = multiplyIt(x = 3, multiplier = 2)
val answer = multiplyIt(multiplier = 2, x = 3)
val answer = multiplyIt(3)
val answer = multiplyIt(x = 3)
```
And all of these will return the same result.

## Class Declarations

Now that we've covered all of that, you can understand the class declaration itself.  

The first thing you'll notice (from the code at top of the article) is that the class declaration looks almost like a constructor.  That's because it is, but in most case you can skip the `constructor` keyword for the primary constructor.

One thing that's different from a Java constructor is that all of the parameters listed automatically become properties of the class.  Just like with Functions, you can declare default values for those properties, if they are not specified in the constructor call.

Primary constructors in Kotlin do not contain any code.  However, you can declare `init {}` blocks in your class and these will be executed, in the order that they appear, from primary constructor.  You can do just about anything that you would expect to do, including set the value for a `val` property in an `init` block.  

If a class implements an Interface, or extends another class, then that can be specified using the type style declaration that was used for functions.

## Visibility Modifiers

While the default visibility for Java is `package-protected`, there is equivalent for this in Kotlin and the default visibility is `public`.  There is also `private`, which does what you'd expect, and `protected` which exposes the class member to any sub-classes.

By default, all functions in a Kotlin class are final, and must be declared with the `open` modifier in order to be overridden by a subclass.

# Language Structure

At this point, you should have enough information to open up a Kotlin class file and navigate around it, understanding at least the structure of the class.  Now let's look at some of the features of the Kotlin language...

## Null Safety

This is the big one!  There is no reason at all to ever get an NPE in Kotlin.  Ever.  

None of the *normal* data types, like `Int`, or `Double` or `String` are allowed to have a Null value.  This means that they must always be initialized when declared, and you cannot put a Null value into them.  

If you want to entertain the idea of having a Null value, then you must declare them as "Nullable".  Nullable types are specified by putting a `?` after the type name.  So `Int?`, `Double?` and `String?` are all types that are allowed to hold Null values.

But note that `Int?` is actually a different type from `Int`.  You cannot perform a mathematical operation on `Int?` directly, nor can you use an `Int?` in an operation to assign to `Int`.  One more time, `Int?` is not `Int`.

In fact, `Int?` is closer to the Java `Optional<Integer>` than anything else.  

In order to use the value in a Nullable type, you need to (effectively) extract it into its non-nullable type.  This is generally done via the `?.` operator, and the `?:` (called the "Elvis operator").  The Elvis operator is a little bit like the ternary operator in Java and, in Java would work like this:

``` java
  int x = (nullableInt.isNotNull()) ? nullableInt.getIntValue() : y;
```
and looks like this in Kotlin:
``` kotlin
  val x : Int = nullableInt ?: y
```
In both cases, `x` would be whatever integer value was in `nullableInt` if it was non-null, and `y` if it was null.

Just as Java's `Optional` has `map()`, Kotlin has `?.run{}`.  The code in the `{}` will be executed with the value passed as a parameter if it's non-null, otherwise nothing happens and the value remains Null.

## Lambdas

Lambdas are much more tightly integrated into Kotlin than Java.  The syntax is similar:

``` kotlin
   val lambda = {param: Int -> some code here}
```
The entire thing is enclosed in the `{}`.  If you aren't going to use the parameter, then you can just replace it with `_`.
``` kotlin
   val lambda = {_ -> some code here}
```   
If the parameter can be inferred, then you can leave out the `param ->` entirely and reference it as `it`.  We'll come back to that shortly.

That's the basic syntax.  One thing that's handy is that if the last parameter in any function declaration is a function, then you can pass it as a "trailing lambda", and if there are no other parameters you can leave out the parentheses entirely.  

So, you can do something like this:

``` kotlin
  val x = doSomething(3, 4){x:Int -> x*2}
     .
     .
     .
  fun doSomething(a: Int, b: Int, func : (Int) -> Int): Int {
    val intermediate = (a*3) - (b *12)
    return func(intermediate)
  }
```
You can also use method references much the same way as in Java.

It's possible to get a little lost between functions declared via `fun` and functions instantiated as lambdas or as variables.  In truth, Kotlin really does treat them very much the same.  For in every instance below, `doubler` is the same:

``` kotlin
  val doubler : (Int) -> Int = {x:Int -> x*2}
  val doubler : (Int) -> Int = {it *2}
  val doubler : (Int) -> Int = {doubleIt(it)}
  val doubler : (Int) -> Int = ::doubleIt
  val doubler = {x:Int -> x*2}
  val doubler = {x: Int -> doubleIt(x)}
  val doubler = ::doubleIt
     .
     .
     .
  fun doubleIt(x: Int) = x * 2
```
The last two versions really make it clear, since you don't need to even specify the type, and `doubler` as just a pointer  for the local function really lays it bare.  

# Extension Functions

It's possible to add a function to a class without creating a subclass.  This is called an ["extension function"](https://kotlinlang.org/docs/extensions.html#extension-functions).  For instance, you can add a function to `Int` to determine if it is even:

``` kotlin
fun Int.isEven() = {this %2 == 0}
```
An extension function is an example of a "receiver function".  A receiver function is one that is automatically passed a parameter which is known inside the function by a standard name, usually `this`.  In the case of a receiver called `this`, if it can be inferred by the compiler the `this` can be left out of a statement when accessing class members (very much like in Java).

# Scope Functions

Scope functions are a special class of "receiver" functions.

There are 5 scope functions, and 4 of them are extension functions.  So we'll look at them first.  

## Transformation Functions (My Term)

These two functions take the object as a parameter and return some other value.  The differ only in only how they name the received object.  They are `let` and `run`.  

Let's look at `run` first:

``` kotlin
val stringLength: Int = "This is a String".run{this.length}
```
That's pretty banal, but you get the idea. The received object is referred to as `this`.  You should also note that when it's clear, the `this` can be inferred by the compiler and left out of the code...

``` kotlin
val stringLength: Int = "This is a String".run{length}
```

Now, `let`:
``` kotlin
val stringLength: Int = "This is a String".let{it.length}
```
You can see that they are pretty much the same, except for how the received object is referred to.

## Configuration Functions (Also, My Term)

These two functions take the object as a parameter and return it back, allowing you to configure an item without instantiating it as a variable.  They are `apply` and `also`.  

I use `apply` all the time with JavaFX, as lots of JavaFX `Nodes` require pretty standard set-up which, in Java, requires instantiating them as a variable just to do something simple:

``` kotlin
hBox = HBox(5, Label("Hello").apply{styleClass += "label.prompt"})
```
Once again, the `this` can be inferred by the compiler and left out of the code. For those not familiar with JavaFX this is the equivalent of `this.getStyleClass().add("label-prompt")`, as `getStyleClass()` returns a `MutableList`.

Still though, that can get ugly, but the same structure can be moved to a function:

``` kotlin
hBox = HBox(5, promptLabel("Hello"))
   .
   .
   .
private fun promptLabel(text: String) = Label(text).apply{styleClass += "label.prompt"}
```
Which is a structure I use all the time.

## The Non-Receiver Function - `with`

If you're old enough to remember the Pascal language, then you might be familial with `with`.  It's a great way to clean up a block of code that has many references to members of a single object.  Whatever parameter passed to `with` becomes `this` in the associated lambda and, of course, the `this` can then be omitted in references to that objects members.  So you can do something like this:

``` kotlin
val theResult: String = with(myClass) {
  name = "Fred"
  val x = age + 20
  doSomething("abc")
}
```
Where `name`, `age` and `doSomething()` are members of `myClass`.

Since `with` is  a function, it can return a value, in this case the result of `MyClass.doSomething()`.

## The Non-Extension Version of `run`

This version of `run` is used when a situation requires an expression, but you want to put multi-line code instead.  The format is `run {}`.  There is no receiver object in this case.

# Language Structure

## String Templates

You can use templates to create strings:

``` kotlin
val x: Int = 123
val string: String = "There are $x carrots"
```
The value of `string` will be "There are 123 carrots".

## If Statements 

For the most part, `if` works exactly as in Java.  However, `if` can also be used as an expression that returns a value.

``` kotlin
val x = if (y > 30) 23 else 100
```
is valid and will assign 23 to `x` if `y` is greater than 30, otherwise `x` will be assigned 100.

For this reason there is no Ternary operator in Kotlin.  This is just about the only case where Kotlin is a little bit more verbose than Java.  However, `if` as an expression is used frequently when the branches are blocks of code.  

## When Expressions

The `when` expression is very similar to the new form of the Java `switch` statement.  Like `if`, `when` can be used as either a statement or an expression.  

## For Loops and Range Expressions
