---
title:  "Kotlin - Should You?"
date:   2021-12-11 10:33:44 -0500
categories: kotlin
logo: /assets/logos/Kotlin.png
permalink: /kotlin-should-you
excerpt: Kotlin has been described as, "The language that Java would have been if it had been designed 25 years later".  It's starting to pick up popularity, and has had a boost from being endorsed by Google for Android development.<br><br>But you're not an Android developer, so should you learn, and use, Kotlin?
---

Kotlin has been described as, "The language that Java would have been if it had been designed 25 years later".  It's starting to pick up popularity, and has had a boost from being endorsed by Google for Android development.

But you're not an Android developer, so should you learn, and use, Kotlin?

### If you are a Java programmer and ...

- You're fed up with Java boilerplate and verbose syntax
- You think that Lambdas are the greatest thing since Generics
- You think that Streams are the greatest thing since Lambdas
- You think that Optional is cool, but at the same time awkward and clumsy
- You want to move towards more functional programming

Then Kotlin is probably for you.  And if you're a Java programmer who isn't all of those things above, then maybe Kotlin would be a good way to expand your horizons and learn about new ways to program.

## It's Not a Steep Learning Curve

Kotlin is very closely related to Java, and there's almost nothing that you can do in Kotlin that you couldn't do in Java as well.  So this means that the concepts which are central to Kotlin aren't going to be all that foreign to you if you come from Java.  This is especially true if you've really become comfortable with lambdas, streams and optional; as all of these concepts are central to Kotlin.  Unlike Java, where these features were added on decades after the language was invented, Kotlin was designed with them as an integral, integrated part from the very beginning.

It probably takes just a couple of hours to learn enough about Kotlin to be able to read and understand most of the Kotlin code you're likely to encounter in real life.  And it seems to take just a few days of working in Kotlin to became comfortable writing new code in Kotlin.   

If you are using Intellij Idea as your IDE, there's even a menu item to convert your Java classes to Kotlin.  That's a great start.  It doesn't produce great Kotlin code, and it definitely looks like Java code written Kotlin, but it doesn't take too much effort to polish up the converted code so that it looks a lot more like it was written in Kotlin from the start.

I've found that you can just take all of the concepts and approaches you'd use in Java and just write them in Kotlin and you'll get a working program.  Then if you go over your code, looking for the bits that seem awkward and clumsy, you can refactor them by researching the concepts with web searches until they look natural.

## What's so Great about Kotlin, Anyways?

### Less Code is Better Code

Over decades of programming I've come to understand that, all other things being equal, the less code you have to write, the better.  Why?

- There's less code to read, so less code to understand.
- There's less places for a bug to hide in.
- Only the bits that make your code unique need to be written.

Kotlin helps you to write less code.  It strips away a LOT of the extra coding that you need to do with Java that, to be honest, no one reads anyways.  

#### Let's Look at Fields as an Example

In Java we have Fields (or "Instance Variables", if you like) in our classes.  But we are taught (correctly) that fields are part of the private inner workings of a class and should not be exposed to the world.  Exposing them would tie our inner class design down to those client classes that are peeking into our class and making decisions based on its internal design.

So we make all of our fields private, and we create two public methods for each field, a getter and a setter.  These methods are named following a convention which puts the word "get" or "set" in front of the field name, with the first letter capitalized.  Typically, it would look like this:

{% gist a55821e44ddb909e290a05491bfaad4f %}

Should you decide to change the way that `intField` is stored, maybe store it as a String, you can keep the external interface constant by providing the conversion operations as part of the getter and setter for that field.

Now, Kotlin was developed knowing that this is the best way to share your class's data without exposing its inner structure to the outside world.  So Kotlin doesn't have "Fields", but it has "Properties".  Think of a Property as an object wrapper around a field which has its own getter and setter.  Kotlin automatically creates the class level getter and setter for the field, delegating them to the getter and setter of the encapsulating property.  The default getter and setter for a property just do what you would expect them to do.

{% gist aee9580aad1e0d3a7347eb742f82ddda %}

You can see that it takes just 1 line of code to do what takes about 8 in Java.

But it gets better!  No one would actually write Kotlin code like that.

In your Kotlin code, you can replace direct calls to the getter and setter with direct references to the property.  It will then invoke the appropriate getter or setter for you.  Additionally, the setter is defined as an operator function, so you can just put the property reference on the left side of an "=" operator and it will turn it into a call to the setter for you.  Like this:

{% gist f0df4dbfb6a5835e093c2983223fb394 %}

As a Java programmer this looks scary!  It looks like it's doing exactly what we were all taught to never, never, ever do - expose the field and directly access it from an external class.  But it's not.  It's following all of the rules, to the letter, but just hiding all of the boilerplate so that it doesn't get in the way.  

The ironic thing is that the resulting code looks exactly the way you would have written your Java before somebody told you not to!

### Mutable, Immutable and Nullable

Variables in Kotlin are intialized with the "val" or "var" keyword and only allowed to be null when explicitly declared as nullable (with a "?" at the end of the type).  Variables declared with the "val" keyword are essentially the same as "final" variables in Java.

Kotlin does everything it can to encourage you to use "val" over "var" whenever possible.  So much so that "var" could be considered to be a "code smell" in Kotlin.  What's the big deal?

> Using vals in your code makes you think about alternative, immutable, functional code. […] Removing vars leads to refactoring. The refactoring leads to new coding patterns. New coding patterns leads to a shift in your approach to programming. This shift in approach leads to transformative code that has fewer defects and is easier to maintain.
>
>    — Beginning Scala, David Pollack

I have found this to be true.  When I look at a block of code and think, "This seems clumsy...wrong", it's usually got a "var" in it somewhere.  Flipping it around and finding an approach that solves the problem without using "var" often results in something that is elegant and clean compared to my original code.

Nullable variables are everything that `Optional` in Java should be ... without the clumsiness of `Optional`.  It's an integral part of the language that you need to master, not an add-on capability that few people understand properly.  

Generally speaking a variable gets a null value in one of two ways, it's a `var` initialized to null which never gets updated to a real value, or it's a `val` instantiated via a function that returns a nullable value.  It's declared with a "?" at the end of the class name, like "String?", or "Int?" or "CustomClass?".  Just like `Optional`, to get that value out of the variable, you need to provide a way for your program to deal with possibility that there's nothing there.

{% gist 5059ba75f5b94a77e2ea406f0814f24d %}

The assumption is that the external database function returns a nullable `Customer` value.  Both functions do the same thing.  The second shows how the `val` variables can actually collapse into just function calls.  

The "Elvis Operator" ("?:") is used like the `orElse()` method of `Optional`.  If you want to do something like `Optional`'s `map()` function, you use the "safe operator" ("?.") and then invoke some method of the variable.  In the first function, `let` is an external "scope" function which takes a lambda argument, passing the value to it as `it`.

Kotlin also has an `!!` operator, which works in the same way as `Optional.get()`.  Using `!!` is generally considered an anti-pattern, and is the only way you can get an NPE with Kotlin.

### Scope Functions

How many times have you instantiated a variable, just to perform some configuration on it by calling it's methods, and then passing it on to some other class as a parameter?  Something like this:

{% gist fdbc1af6c4b89a7bf1ac17342961d67e %}


But Kotlin has an `apply` scope function which takes a lambda which receives the value as `this` and returns `this`.  Inside the scope function, you can treat the `this` as implied when it makes sense.  So that code above becomes:

{% gist b05ceb242a9f98d350404bf6d07dc752 %}

So now we don't even need a variable reference to the object at all, we can just pass the results of the constructor through the `apply` and directly off to another object's method.  In a way, you can use the `apply` to turn a class's configuration methods into decorators.  

There are 5 different kinds of scope functions in Kotlin.  You can read about them  [here](https://kotlinlang.org/docs/scope-functions.html) .

By the way, if the multi-line lambda breaks up the flow of your code a bit much, you can very easily turn it into a named function in Kotlin:

{% gist 1029617ae1412e59a021ab82584942ba %}

### There's a LOT More

Any attempt to list all of the features of Kotlin, and how they improve upon kludgy Java implementations would just result in a TLDR blog.  What you really need to know, though, is the Kotlin takes virtually all of the modern concepts added to Java and integrates them right into the language so that they feel natural and spread everywhere.  

Lambdas are a really good example of this.  You can get away without lambdas in Java, but you couldn't write much of anything in Kotlin without them.  They're a crucial concept to the language, and using them in Kotlin feels completely natural.


## What Are the Downsides to Using Kotlin?

From a technical perspective, there are very few.  You can freely co-mingle Java and Kotlin in you projects, and you can call any Java library from Kotlin.  With a very small amount of care, any of your Kotlin code can be called from Java, and you can use any Kotlin libraries with Java.

Kotlin compiles to run in the JVM, and you can tell the Kotlin compiler to generate code compatible with whatever version of the JVM you want.  So it's easy to maintain compatibility between your Java and your Kotlin code in your project.

Although Kotlin has been around for almost a decade now, it's nowhere near as old as Java, and doesn't have the wealth of resources that you are going to find with Java.  The programmer community is smaller, so there just aren't as many people to answer your questions on sites like StackOverflow.com, and you're less likely to find that somebody has already asked your question so the answer is just there waiting for you.

Another thing you'll need to deal with is that, even though Kotlin can use just about any Java library you might find, virtually all of the examples and the documentation are going to be in Java.  You'll probably also find that any questions and answers you find on sites like StackOverflow.com are going to be in Java, too.

It's also possible that some of the tooling that you're using right now with Java can't be used with Kotlin, and you may not be able to find a Kotlin compatible replacement.  This will undoubtedly improve over time.

If you are working on a preexisting code-base with a lot of old Java code, there are going to be practical issues around converting everything to Kotlin.  Especially in the absence of a comprehensive test suite.  But if you're opening up a class for refactoring or modification, you might have the opportunity to convert it to Kotlin in the process.  The resulting code will almost certainly be smaller, cleaner and easier to extend and maintain.

#### A Final Thought

One thing about learning Kotlin, though; it will change you.  A phrase that you'll hear a lot is "idiomatic Kotlin", which means the programming equivalent of "speaking like a native".  You don't hear about "idomatic Python" or "idomatic BASIC" too much, but it seems like "idiomatic Kotlin" is a key concept of learning the language.  

There's something about Kotlin, maybe its focus on fixing the bad parts of Java, that makes your bad Kotlin code stick out like a sore thumb.  You see it, and you can't help but think, "That just doesn't look right!".  And, generally speaking, fixing that bad code means writing it in a more idiomatic Kotlin way.  

Do that often enough, and that idiomatic Kotlin way becomes *your* way to write code, and it changes how you approach writing code.

And then you have to go back to Java to work on some legacy code...
