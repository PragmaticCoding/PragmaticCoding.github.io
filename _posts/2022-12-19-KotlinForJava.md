---
title:  "Kotlin For Java Programmers"
date:   2022-12-19 12:00:00 -0500
categories: kotlin
logo: /assets/logos/Kotlin.png
permalink: /kotlin/kotlin_for_java_programmers
excerpt: Need to understand a Kotlin program, but you only know Java?  This article should give you everything you need to know to get started.
---

# Introduction

The first time I ever looked at a Kotlin program was when one of my developers found a cool JavaFX library called ["DirtyFX"](https://github.com/thomasnield/DirtyFX).  We wanted to understand it worked, but when we looked at the source code on GitHub we were stymied because it was in Kotlin.  

It seemed impenetrable.

Years later, I've gone back to Kotlin and learned it.  It's awesome.  And, if you're a competent Java programmer, you can learn enough to start using it with a 4 hour YouTube tutorial.  Then it takes about a week or so to get comfortable, and somewhat longer to get proficient at it.  

This article isn't designed to replace a 4 or 5 hour YouTube tutorial.  It's intended to give a typical Java programmer enough information to understand a typical Kotlin program in a much shorter period of time.  It's also intended to give a typical Java programmer a feel for how Kotlin works, and a start to understanding why many Kotlin programmers think it's the way forward in the JVM world.

I've also provided links to the Kotlin official documentation as part of the text.  So if you want to look into some aspect of Kotlin a little more deeply, it should be easy for you to do so.  The Kotlin docs are really well written and easy to understand, too.

One more thing:  If you study Kotlin for any length of time, you'll come across the term "idiomatic Kotlin".  This is the idea that while Kotlin shares the JVM, just writing code the way you would in Java but in Kotlin isn't going give you something that *feels* like Kotlin.  There's definitely a Kotlin approach to coding, and coding in that way gives you "Idiomatic Kotlin".  In order to *write* code this way, you do really have to have a solid grasp of a lot of Kotlin, but once you get used to some of the quirky things you'll see in Kotlin code you should be able to easily read and understand it.

## Why Kotlin

When I learned Kotlin I realized that there was lots of stuff in Java that bugged me (or should have) that Kotlin just fixes in a seamless and natural way.  Null safety is a clear example of that.  Sure, in Java you can use `Optional`, but Kotlin's approach is nicely integrated into the language in such a way that Null safety is virtually mandatory in any code you write.  From what I've seen, just about every Java programmer who takes the time to learn enough Kotlin to become somewhat proficient in it wishes that they could switch over to Kotlin full time.  

In general, Kotlin code is shorter and more intuitive than the equivalent in Java, and is therefore easier to read and understand.  One example you'll see here is how Kotlin eliminates the need for writing getters and setters, while providing the same functionality and separating the public interface from the internal workings of your class.  Then Kotlin goes one step further by interpreting external references to member properties as calls to the getters and setters.  In the end, Kotlin allows you to write code exactly the way that you always wanted to, and does all the work, behind the scenes, of the boilerplate code you have to write in Java.

# Class Structure

Let's start at the top, class structure.  We'll use this sample code to talk about a few topics:

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

Kotlin basically supports all the same [types](https://kotlinlang.org/docs/basic-types.html) as Java, although the primitive types aren't used.  So anything that would be `int` or `Integer` is `Int` in Kotlin.

The second thing that you'll notice is that type declarations are of this structure:

``` kotlin
keyword name : type
```
This structure is the same whether you're declaring a field, a variable, a class or a method.  

If you are initializing a variable and it can be reasonably inferred as to what it is, then you do not need to explicitly specify the type:

``` kotlin
var abc = 5
```
Will give you `Int`.  However, if you want `Long`, then you need to specify it unless the initial value is too large for `Int`:

``` kotlin
var abc : Long = 5
```

The same structure follows when you want to define a variable as an interface when the initialization calls the constructor of an implementation of that interface.

## Instance Variables

Here, we're very specifically using the term "instance variables" because Kotlin has a different structure for static elements, and these instance variables are very different from Java fields.  Kotlin calls instance variables, ["Properties"](https://kotlinlang.org/docs/properties.html).

A Kotlin property is actually a structure that is backed by a "field".  There is a default getter and setter for the property, and should you chose to override them you can access the backing field inside your getter/setter code.  For now, this is all you need to know about this.

One advantage to Kotlin is that it automatically invokes the getter or setter for a property when it is directly accessed via dot notation in your code.  Let's look at this snippet:

``` kotlin
myClass.property3 = myClass.property4[2]
```
This is completely proper and valid Kotlin code.  It calls the setter for `property3` and the getter for property4.  On top of that, it calls `List.get()` by using `[]` construct.  You can also do this:

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

Methods in Kotlin are called [Functions](https://kotlinlang.org/docs/functions.html) and declared using the `fun` keyword.  The return type of a function is declared the same way as for a variable and the body of the method is enclosed in `{}` if there is more than one line.  If the function body is an expression, then you can just use `=` and put the code.  For instance:

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

Now that we've covered all of that, you can understand the [class declaration](https://kotlinlang.org/docs/classes.html) itself.  

Looking at the code at top of this article, you'll notice that the class declaration looks almost like a constructor.  That's because it is, but in most case you can skip the `constructor` keyword for the primary constructor.

One thing that's different from a Java constructor is that all of the parameters listed as `val` or `var` automatically become properties of the class.  Just like with Functions, you can declare default values for those properties, if they are not specified in the constructor call.

Primary constructors in Kotlin do not contain any code.  However, you can declare `init {}` blocks in your class and these will be executed, in the order that they appear, from the primary constructor.  You can do just about anything that you would expect to do, including initializing the value for a `val` property in an `init` block.  

If a class implements an Interface, or extends another class, then that can be specified using the type style declaration that was used for functions.

Static elements are created by including them in a "[Companion Object](https://kotlinlang.org/docs/object-declarations.html#companion-objects)" inside the class.  

## Visibility Modifiers

While the default visibility for Java is `package-protected`, there is no equivalent for this in Kotlin and the default visibility is `public`.  There is also `private`, which does what you'd expect, and `protected` which exposes the class member to any sub-classes.

By default, all functions in a Kotlin class are final, and must be declared with the `open` modifier in order to be overridden by a subclass.

# Language Features

At this point, you should have enough information to open up a Kotlin class file and navigate around it, understanding at least the structure of the class.  Now let's look at some of the features of the Kotlin language...

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

You'll also see the term "higher order function" in a lot of documentation.  This is simply a function that takes another function as a parameter.  It's far more common in Kotlin than Java...

## Functions as Data

This is probably one of the biggest features of Kotlin that should have a impact on the "feel" of programs written in Kotlin.  Kotlin makes it very easy to pass snippets of code around from place to place, very much like data.  Java has headed a little bit in this way with the introduction of "Functional Interfaces" and lambdas, but Kotlin bakes the concept in from the beginning.  

First off, you can declare a function as a data type just by specifying its inputs and outputs, in a format that looks a little bit like a cross between a lambda and a generic type declaration.  For instance:

```kotlin
var abc : (String, Int) -> Double
```
Here, the variable `abc` is declared to be a function that takes a `String` and an `Int` as input and returns a `Double`.  There's no need to declare a Functional Interfaces like `Function`, `Predicate` or `Consumer` as you do in Java.

You can even declare a "Type Alias" to give a name to your function type:

```kotlin
typealias NumberFinder = (String, Int) -> Double
var abc : NumberFinder
```

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

Finally, you can treat `doubler` from above just like `doubleIt()` in your code:

```kotlin
val y = doubler(abc)
val j = doubleIt(abc)
```
Both would work just fine.

## Extension Functions

It's possible to add a function to a class without creating a subclass.  This is called an "[Extension Function](https://kotlinlang.org/docs/extensions.html#extension-functions)".  For instance, you can add a function to `Int` to determine if it is even:

``` kotlin
fun Int.isEven() = {this %2 == 0}
```
An extension function is an example of a "receiver function".  A receiver function is one that is automatically passed a parameter which is known inside the function by a standard name, usually `this`.  In the case of a receiver called `this`, if it can be inferred by the compiler the `this` can be left out of a statement when accessing class members (very much like in Java).

## Scope Functions

[Scope Functions](https://kotlinlang.org/docs/scope-functions.html) are a special class of "receiver" functions.

There are 5 Scope Functions, and 4 of them are Extension Functions.  So we'll look at them first.  

### Transformation Functions (My Term)

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
You can see that `run` and `let` are pretty much the same, except that the received object is referred to as `it` in `let`.  When a Scope Function uses `it` you can use the normal lambda syntax to change it to something more meaningful if you like.

### Configuration Functions (Also, My Term)

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

### The Non-Receiver Function - `with`

If you're old enough to remember the Pascal programming language, then you might be familiar with `with`.  It's a great way to clean up a block of code that has many references to members of a single object.  Whatever parameter passed to `with` becomes `this` in the associated lambda and, of course, the `this` can then be omitted in references to that objects members.  So you can do something like this:

``` kotlin
val theResult: String = with(myClass) {
  name = "Fred"
  val x = age + 20
  doSomething("abc")
}
```
Where `name`, `age` and `doSomething()` are members of `myClass`.

Since `with` is  a function, it can return a value, in this case the result of `MyClass.doSomething()`.

### The Non-Extension Version of `run`

This version of `run` is used when a situation requires an expression, but you want to put multi-line code instead.  The format is `run {}`.  There is no receiver object in this case.

## Null Safety

[Null Safety](https://kotlinlang.org/docs/null-safety.html) is one of the biggest features of Kotlin!  There is no reason at all to ever get an NPE in Kotlin.  Ever.  

As important as it is, it had to be left until this point because you need to understand how lambdas and Scope Functions work first, which shows you how tightly integrated into the language null safety is.  

None of the *normal* data types, like `Int`, or `Double` or `String` are allowed to have a `Null` value.  This means that they must always be initialized when declared, and you cannot put a `Null` value into them.  

If you want to entertain the idea of having a `Null` value, then you must declare them as "Nullable".  Nullable types are specified by putting a `?` after the type name.  So `Int?`, `Double?` and `String?` are all types that are allowed to hold `Null` values.

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

Just as Java's `Optional` has `map()`, Kotlin has `?.{}`.  The code in the `{}` will be executed with the value passed as a parameter if it's non-null, otherwise nothing happens and the value remains Null.

```kotlin
val x : String? = getSomething()
val y : Double? = x?.length()?.run{
  val abc = this * 3
  return abc / 7.0
}
.
.
.
fun getSomething(): String? {}
```
In the expressions for `y` the code after `?.` is only executed if the value is non-null.

# Language Structure

This is a quick survey of some of the common language elements that are used in Kotlin that are different from Java.

## String Templates

You can use templates to create strings:

``` kotlin
val x: Int = 123
val string: String = "There are $x carrots"
```
The value of `string` will be "There are 123 carrots".  For more complex expressions, use {}.  Also, `println` is part of the standard library, so there's no need for `System.out.println()`...

```kotlin
println("There are ${fridge.carrots + table.carrots} carrots")
```

## Equals

No surprise here, in Kotlin "==" does what you've always wanted it to do in Java!  "==" invokes the `equals()` method for any non primitive class.  If you want referential equality, use "===".

## If Statements

For the most part, [if](https://kotlinlang.org/docs/control-flow.html#if-expression) works exactly as in Java.  However, `if` can also be used as an expression that returns a value.

``` kotlin
val x = if (y > 30) 23 else 100
```
is valid and will assign 23 to `x` if `y` is greater than 30, otherwise `x` will be assigned 100.

For this reason there is no Ternary operator in Kotlin.  This is just about the only case where Kotlin is a little bit more verbose than Java.  However, `if` as an expression is used frequently when the branches are blocks of code.  

## When Expressions

The [when](https://kotlinlang.org/docs/control-flow.html#when-expression) expression is very similar to the new form of the Java `switch` statement.  Like `if`, `when` can be used as either a statement or an expression so it can return a value.  When the subject of a `when` statement is an `Enum` or a `sealed` class, then the branches of the `when` must be exhaustive, or include an `else` branch.  

## For Loops and Range Expressions

In Kotlin [for loops](https://kotlinlang.org/docs/control-flow.html#for-loops) always work across a collection.  To increment a value over a range, Kotlin has a type of `Collection` called a `Range`.  The Java code:

```java
for(int x = 0; x < 6; x++) {}
```
would become:

```kotlin
for(x in 0..5) {}
```

If you want to iterate over a `Collection` but also have an index, there's a way to do that, too:

```kotlin
for((index, item) in collection.withIndex()){
  println("Item # $index is ${item.description}")
}
```

## Collection Operations and Sequences

In Kotlin, you can do almost all of the operations you would do with `Streams` directly on any `Collection`.  This means that there's no need to perform `.stream()` on a `List` or any other `Collection` most of the time.  Operations performed on a `Collection` create a new `Collection` with the elements transformed in some way.

When you do want to process in a manner similar to Java `Streams`, Kotlin has `Sequences`.  You can use the `Collection.asSequence()` function to do this.  There are some cases where the performance of a `Sequence` might be better than a series of `Collection` operations.  There are also some function that are only available as `Sequence` operations.

You can also create an infinite `Sequence` with the `generateSequence()` function that specifies a starting value and a function to create the next value.  You can then treat it like any other `Sequence`.  `Sequence` has a function called `take(x)` that allows you to take the first `x` values of a `Sequence`.

## Maps and Pairs

[Maps](https://kotlinlang.org/docs/map-operations.html) in Kotlin are made up of tuples, just as in Java, but they are thought more of as a collection of tuples than key/value containers.  Tuples are constructed and then added to the `Map` as in the following code:

```kotlin
val testMap = mutableMapOf("x" to 27, "abc" to 55, "hello" to 74)
testMap.put(Pair("fred", 18))
```
Additionally, you can use the `[]` shorthand operator instead of `put()` and `get()` on a `Map`

```kotlin
testMap["george"] = 100
println(testmap["fred"])
```
