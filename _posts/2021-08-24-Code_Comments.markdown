---
title:  "Are Your Code Comments Helping Anyone?"
date:   2021-08-24 10:33:44 -0500
categories: programming
logo: /assets/logos/LittleBrain.jpg
permalink: /are-your-code-comments-helping-anyone
excerpt: Comments in code are supposed to help other programmers understand your code so that they can build on it, improve it and, sometimes, fix it.  Are the comments that you're leaving in your code helping anyone to do that?
---


Comments in code are supposed to help other programmers understand your code so that they can build on it, improve it and, sometimes, fix it.  Are the comments that you're leaving in your code helping anyone to do that?

For nearly 40 years now, I've made my living as a programmer, and that means reading other people's bad code.  Including my own.  The truth is that most of us won't always have the luxury of "greenfield" programming and that means that, sooner or later, you're going to have to do maintenance work on some code that someone else wrote years ago.

I've done this a lot.  I'm good at it.  I've probably looked at millions of lines of other people's code, trying to figure out what it does, how to fix it, and how to enhance it.  It can be hard - harder than writing new code - and I've known quite a few good programmers who never managed to master it.  For a lot of my career I've been the on-site expert on the systems, and there hasn't been anybody to go to if I couldn't figure something out.  So I've had no choice but learn how to read other programmers' code, and to understand what it's doing and how it's doing it.

## I'm Probably Going to Delete Your Comments

The first thing I do when I see a program with **a lot of comments** in it is to delete the comments.

Why would I do that?

Well, first of all, I'm not trying to be mean about it, or disrespect the programmer who took the time to write all of the those comments that they were sure would be helpful later on.  When open up a program to do some maintenance on it I'm extremely focused on what I need to do.  I'm not interested in critiquing somebody's code, or making a philosophical point.  I'm just there to understand it just enough so that I can do what I need to do.  Sometimes, that means doing cleanup on the code so that it's in a state that I can work with it efficiently.  And that may mean deleting the most, if not all, of the comments.

Most importantly, the comments get in the way.  They take up screen space that reduces how much executable code I can see at once.  That's a big factor when you're trying to understand some complicated logic or follow some spaghetti code.  For instance, I like to be able to see an entire "if" statement all at once - the condition, along with both of the "then" and "else" clauses.  If the comments are causing the last lines of the "else" clause to drop off the bottom of the screen, there's an easy way to fix that.  Delete the comments.

Beyond that, I don't trust comments.  Nobody should.  By definition they don't execute, so there's no guarantee that they are correct.  What if the program has changed since they were written and nobody bothered to update the comments?  What if the comments were never correct?  The truth is that you don't know, you can never know, and you're better off understanding the code itself than relying on dubious comments - and all comments are dubious.

When I really need to understand some complicated program code, I usually do the following:

- Delete the comments
- Delete any commented-out code
- Reformat the code so that it's easier for me to read
- Rename variables and methods that aren't clear
- Pull giant blocks of code out into their own methods
- Split long conditions and expressions into smaller parts and assign them to variables
- Put brackets around clauses in conditions and long expressions
- Fix anything else that confuses me

Unless I'm really reworking an entire program, I probably won't commit any of these changes back into the code base.  The idea is just to get the code into a state where I can understand it, then I'll usually revert those changes and make my fix or enhancement to the original version.  I'm not looking to turn a small fix into a testing nightmare.

You may disagree with all of this.  Fair enough, but it's a safe bet that any programmer who's had a lot of experience in maintenance programming, and who's good at it, would have developed a system something like this.  They're certainly not going to read your comments.  And if we're not going to read your comments, why are you putting them in your code?

## If Not Comments, Then What SHOULD You Do?

Basically, you should write your code so that it looks like it would after I've done the things listed above.  Really though, what this means is that you should write your code so that it's clear and easy for anyone in the future to understand what it's doing without comments.

This often boils down to developing two crucial skills: naming things, and putting your code in the right place.

### Naming Things

For some reason, naming things is super hard.  People have even invented schemes like "BEM" to help programmers figure out how to name stuff.  Following rules can help, but the only thing that really matters is that the particular name of a particular thing in a particular context makes it easy to understand what the code does.

There's really only one rule about naming things in programs:  **Pick the simplest name which makes it easy for someone to understand the most important thing about what an element is, and how it will be used.**

Here are some things to think about when naming anything:

- **Names Should Be Meaningful**

    A name should explain what something is, what it's for or what it does.  It needs to be only specific enough to differentiate it from other things like it.  As an example; you might have a method for retrieving a customer record from a database using an account number that HAS to return a value (like if the account number is referenced in an invoice).  You could call it "fetchCustomerRecord".  But you might have a different method that returns on Optional of the customer record for those cases where the account number might no be on file (if it comes from a user typing in an account number).  You might call that method, "fetchCustomerRecordIfPresent".  

- **Names Should Be Simple**

    Never forget that people need to be able to read your code, and people have trouble coping with long or complex names for things.  As soon as a name starts to look like machine code, people's brains tend to stop processing them.  So, if possible, don't use leading or trailing underscores.  Don't use dollar signs, or percentage signs or carets or other weird, meaningless symbols in your names unless you have to.

    Decide if you're are going to use acronyms to help keep the names short, decide whether or not you want them to be all upper case inside your names and then be consistent.  Acronyms can be OK if they're commonly understood in the business domain and aren't going to cause confusion in your code base.  All caps can be problematic if you are using camel case in your names, as they can obscure the beginning of the next word.  

    Names should only be as long as they need to be.  Longer names are problematic with formatting your code, and that can be a significant barrier to people understanding your logic.  

- **Context and Scope Are Important**

    Generally speaking, a name only has to make sense within the scope and context that it's going be used in.  For instance, using a variable name like "results", or "returnValue" in a method that returns a value is generally helpful, even though it doesn't appear to describe what's held in the variable.  That's because it describes the most important thing about that variable in that context - that this is the value that's going to be returned by the method.  If I look at a method and see variable instantiated on the very first line called, "results", I know that this is the element I have to pay close attention to.

    In the same vein, private methods in a class can have names that only make sense within the context of other methods in the class.  That's because the name of the method is intended to explain what's happening in the code that calls it.  So a method name like, "createModelFromInvoice", can be fine because in any place that it might be called, "model" and "invoice" are probably going to be clearly understood.

### Putting Your Code in the Right Place

When code is in the right place, it's so much easier to understand.  

Even more importantly, **when code is in the right place, often you won't even need to understand it**.  Why?  Because when it was in the wrong place, it was right in the middle of the code you did need to understand.  Once it's moved to somewhere else and properly named, you may not even need to look at it any more.

#### The Single Purpose Principle

Every chunk of code that you write should do one thing, and one thing only.  That "chunk" might be a block of code, a method or a class.  A good way to think about this is to try to have each "chunk" designed so that it can be described completely in a simple phrase.  If the thing that it does is complicated, then you can probably divide it up into smaller pieces, each of which in turn can be described completely in a simple phrase.  Repeat as necessary.

As an example think about a method called `calculateTotalSale()`, which should have a simple description of "calculate the total sale amount".  So far so good.  But if part of that method has to:

- determine taxes by looking up the state
- look up the tax rate
- separate out the taxable from non-taxable items
- calculate shipping
- calculate handling charges
- total it all up

Now "calculate the total sale amount" doesn't **completely** describe the method, does it?  So the best thing might be to pull the sales tax code out of `calculateTotalSale()` and put it into its own method, `calculateSalesTax()`, which "calculates the sales tax".  But does it just "calculate the sales tax"?  Not really.  It's probably best to pull out the state lookup code into a method called `fetchStateTaxRate()`, and then create another method called `determineTaxableItems()`, and so on.  What's left in `calculateSalesTax()` is just the logic that puts it all together, calling the other methods, which each have a single purpose, to do the work that their names describe.

And now, when someone comes along and needs to implement some kind of change, maybe a new rounding method, they don't need to understand all the details of how sales tax rates are determined, because it's not sprinkled in with all of the other code that you do need to look at to make the change.

#### The Information Expert Principle

The Information Expert Principle is one of the patterns defined in [GRASP](https://en.wikipedia.org/wiki/GRASP_(object-oriented_design)).

> Problem: What is a basic principle by which to assign responsibilities to objects?
> Solution: Assign responsibility to the class that has the information needed to fulfill it.

It's not unusual to see code like this:

```
    int thresholdValue = 12;    
    .
    .
    .
    double answer = 0d;
    if (object.getSomeValue() > thresholdValue) {
       answer = (object.getValue1() + object.getValue2()) / object.getValue23();
    } else {
       if (object.isSomething()) {
          answer = object.getValue22() * object.getValue17();
       else {
          answer = 87.45;
       }
    }

```  

Everything after the setting of the the thresholdValue uses information from `object`, and probably should be the responsibility of `object` to calculate.  It shouldn't be where ever it currently is sitting.

Far better would be to create a method in whatever class `object` is and pass it the threshold as a parameter.  Something like this:

```
public double calculateSomething(int thresholdValue) {
    double answer = 0d;
    if (someValue > thresholdValue) {
       answer = (value1 + value2) / value23;
    } else {
       if (isSomething()) {
          answer = value22 * value17;
       else {
          answer = 87.45;
       }
    }
   return answer
}
```
And then the original code would be refactored to:

``` java
int thresholdValue = 12;    
.
.
.
answer = object.calculateSomething(thresholdValue);
```

There's also a good chance that three other things will come out of this refactoring:

- Some or all of the getters like object.getValue22() will no longer be needed and can be deleted.
- The logic from `calculateSomething()` is duplicated elsewhere in the system and can be replaced with a call to `calculateSomething()`
- `calculateSomething()` is now testable, all by itself.

But most important is that the logic that is now in `calculateSomething()` is not cluttering up the other code any more.  It's just one line `answer = object.calculateSomething(thresholdValue)`.  And, of course, it's not called "calculateSomething", it's got a meaningful name which describes it properly, so that when you read it you can tell at a glance what it does.  That's an important point, because it may not have been clear from that block of code what it was trying to do.

So when you read this code, you'll know right away if you care about what `object.calculateSomething()` is doing, and if you need to look into it to finish my task.  Probably, you won't, and you will have saved yourself the effort of opening up what ever class `object` belongs to and verifying that `someValue`, `someValue1`, `isSomething` and all the other fields mean what you think they mean.  You won't have to confirm that there's not some hidden side-effect in that block of code, either.  

##### An Exception to the Information Expert Principle

If you have a class which is a JBOD or some kind of a data model that fits into a framework, you probably won't want to have business (or application, if you like) logic baked into that data class.  Business logic generally belongs in a specific place where you know to go and find it, without having to hunt around in a slew of data classes.

Even if you have to create some relationship between elements in a JBOD, you shouldn't define that relationship inside the class if it comes anywhere close to being business logic.  For instance you could have a `Rectangle` class that has height and width fields.  Maybe you could have a method called `getArea()` because that might be considered sufficiently generic to be included in a data model of that type.  But if you had some business logic that required determining if the height was 2.8 times the width, then adding a `isRectangleHighEnough()` method would probably be a bad idea.

## General Readability Issues

### Creating Local Variables for Readability

Sometimes you'll come across something like this:

``` java
if (theList.get(listIndex).retrieveAValue(xyz) < someValue) {
.
.
.
}
```

It might not be clear from the code what that value means in the context in which it's being used.  Worse even...

``` java
if ((theList.get(listIndex).retrieveAValue(xyz) * 0.754) < otherList.get(index2).retrieveSomething(abc)) {
.
.
.
}
```
That's not just ugly, it's hard to understand, too.

Last example:

``` java
if ((theList.get(listIndex).retrieveAValue(xyz) * 0.754) < otherList.get(index2).retrieveSomething(abc)) {
.
   double something = theList.get(listIndex).retrieveAValue(xyz) / 2;
.
}
```
Here we've used that retrieved value twice, which means that you have to check that code twice to make sure that it really **is** the same thing. What if the second use passed "xjz" to `retrieveAValue()`?  Would you catch it?

Far better would be something like this:
``` java
double xyzValue = theList.get(listIndex).retrieveAValue(xyz);
double somethingRatio = 0.754;
double contextualThreshold = otherList.get(index2).retrieveSomething(abc);

if ((xyzValue * somethingRatio) < contextualThreshold) {
.
   double something = xyzValue / 2;
.
}
```
This is a few more lines of code, but the result is way easier to understand, and it's now perfectly clear that `xyzValue` is used twice.  On top of that, the variable names should clearly state what all of those values are and what they mean in this context.  No comments are required now because the code has been designed to make it clear what's going on.

### Using Variables to Explain Conditions

That last example still has one possible issue.  What's that `xyzValue * somethingRatio`? This can become really important when a condition has multiple clauses, especially cases where logical and's and or's are mixed together.  It's much clearer to break out the clauses as booleans and give them meaningful names.  Something like this:

``` java
boolean hasEnoughMojo = (someValue >= thresholdValue);
boolean isThursday = dayOfWeek.equals(CalendarTools.getWeekDay(4));
boolean isQualified = certificateList.contains(courseName);

if ((hasEnoughMojo || isThursday) && isQualified) {
    .
    .
    .
}
```
This is helpful for two reasons.  First, the `if` statement is now super simple to read.  With the conditions broken out, it's easy to see how the conditions fit together.  Secondly, it's clear what the individual conditions mean - because they have names!

There's an extra bonus value here, too.  Look at that variable `isQualified`.  That name speaks to *intent*.  It asks, "Is it qualified"?  What if, upstream, the API that supplied `certificateList` is changed to include current courses?  Would that lookup function now mean "qualified"?  Without that intent baked into the code, it's impossible to know whether or not there's an issue here.  The way it is, when you realize that `certificateList` now contains certificates in progress, you can go and ask the business whether or not that counts as "qualified".  

### Unnecessary Local Variables:

Sometimes the opposite is required.  I see this all the time:

``` java
public void start(Stage primaryStage) throws Exception {
    primaryStage.setTitle("window name ");
    BorderPane layout = createLayout();
    Scene scene = new Scene(layout, 300, 250);
    primaryStage.setScene(scene);
    primaryStage.show();
}
```
This just takes up space on the screen.  The variables `scene` and `layout` add nothing.  This will work just as well:
``` java
public void start(Stage primaryStage) throws Exception {
    primaryStage.setTitle("window name ");
    primaryStage.setScene(new Scene(createLayout(), 300, 250));
    primaryStage.show();
}
```
Generally speaking, if something is boilerplate then you can assume that any programmer looking at it is going to understand what it does and you should feel free to compress it as much as possible without causing confusion.  I use Intellij's "inline variable" function all the time.

It's a good rule of thumb that less code is going to be easier to understand than more code.  So, when in doubt, write less code.  Don't try to be clever about it - that has the opposite effect - but when something can be done clearly in one line of code, don't do it in two lines.

## In Conclusion

If I open up a program and see that it has just one or two comments, then I'll usually take the time to read those comments and see how they relate to the code.  Because, even if the code isn't stylistically what I like, the fact that there's only a couple of comments is big clue that the programmer understood what comments are for and they are, therefore, worth reading.

Comments do have a useful role when they are employed to explain things that can't adequately be made clear through the executable code itself.  But they really should be a last resort only.

Here's the big benefit from avoiding comments:  **It's harder to write bugs when your first priority is to write clear simple code that is easy to understand without comments.**  It's true.  I don't know how many times I've discovered bugs in old code just by doing all those refactorings I listed up near the top.  Without even getting to the point of even understanding what the code does!

And if you write your code the same way from the start, you'll find those bugs yourself as you are writing the programs.
