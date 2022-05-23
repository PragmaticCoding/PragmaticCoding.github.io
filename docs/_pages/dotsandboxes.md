---
layout: single
title: JavaFX
toc: true
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
permalink: /private/dotsandboxes
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
diagram: /assets/private/BoxesAndLines.png
simulation: /assets/private/gameplay.gif
---

# Designing a Game

Everyone always wants to start at the UI.  After all, it's a visual game, no?  

But that's not right.  If you can't visualize how the game works as data and logic, then having a UI created without that understanding just puts you way, way backwards.  

So the first thing is to think about how the game works as **data**.  

# How I Would Think About It

OK, so yes we need to look at the game visually, but just to translate it into data.  Here's an example of the game layout, with some labels to link to data:

![Diagram]({{page.diagram}})

Now you have horizontal lines and vertical lines.  And, to start with, we'll track them separately.

The important thing to note is that Box(0,0) is bounded by H(0,0), H(0,1), V(0,0) and V(1,0).
And Box(1,1) is bounded by H(1,1), H(1,2), V(1,1) and V(2,1).

So we can generalize from this that Box(x,y) is bounded by:

1. H(x,y)
1. H(x,y+1)
1. V(x,y)
1. V(x+1,y)

Now we have a dependency relationship between the lines and the boxes!

We're done with the visuals for now.  We got what we needed out of it.

## Game Mechanics

Now we have two object types `GameLine` (let's call them that since JavaFX has it's own class called `Line` that we'll probably need to use), and `Box`.

There's really one thing about a `GameLine` that is important: if it's been "activated".  That's it.

For a Box there are two things that are important: If all of the lines associated with it have been activated - then it's "completed", and which player completed it.

It's really tempting to do something like this because the `GameLine` only has a single Boolean attribute:

``` java
private Boolean[][] horizontalLines = new Boolean[4][4];
private Boolean[][] verticalLines = new Boolean[4][4];
```

But that would probably be a mistake.  First off, it's not very Java because it's not very Object Oriented.  Secondly, the knowledge about the GameLine is split between the array itself (which holds its location) and the value in the array.

Better would be to create an object, `GameLine` and give it these fields:

1. Column
1. Row
1. Type (Horizontal or Vertical) (Make this an Enum called `LineType`)
1. Activated? (Boolean)

This is so much better because it's now an autonomous entity.  **Everything about it is encapsulated within it, and you don't need to depend on whatever data structure it's stored in to find out what kind of line it is, or where it's located.**

Once you've done this, you can ditch the arrays completely since the `GameLines` themselves know where and what they are.  So you can just put them in a List:

``` java
private List<GameLine> gameLines = new ArrayList<>();
```

You can find a line by iterating through the gameLines until you get one with the correct row, column and type.

Now you're probably thinking,  "Ooooh, yuk!  That's so much less efficient than just going `line = horizontalLines[37][54]`.".

Maybe so, but...

I had a boss years ago that said this to me when I was scratching my head trying to make something more efficient for no reason: "What do you care if the computer has to work harder?  It's not going to get tired."

That's so true.  Even if your game board was 1000x1000 dots, it's gonna take a loop about 0.000001s to find a line in an array.  It's totally inconsequential.  It's definitely going to be too fast for a user to detect a delay.  So forget about it.

It's way more important to design your application so that it's easier to understand, and messing around with multi-dimensional arrays always gets more complicated that you need it to be.

Moving on...


Boxes have the following fields:

1. Column
1. Row
1. List of GameLines around it.
1. Completed? (Boolean)
1. Owner (player 1 or 2, or none) (Make this an Enum called `BoxOwner` or `Player`)

Same deal as with the `GameLines`, just use a list:

``` java
private List<Box> boxes = new ArrayList<>();
```

## Populating The Board Data

Here's the key:  Treat that `List` of `GameLines` like a cache.  

What does that mean?

Write a method like this:

``` java
GameLine findOrCreateGameLine(LineType type, int column, int row) {
    gameLines.forEeach(gameLine -> {
       if ((gameLine.type == type) && (gameLine.column == column) && (gameLine.row == row)) {
         return gameLine;
       }
    })
    // You can only get here if nothing matched
    GameLine gameLine = new GameLine(type, column, row);
    gameLines.add(gameLine);
    return gameLine;
}
```

This means that a `GameLine` doesn't exist until you go looking for it the first time, in which case it creates it and puts it into the cache.  After that, it'll just be pulled out of the cache when it's needed.

The awesomeness of this is that you never have to do the nested loop stuff to create the lines!  

Think about populating the game board from the perspective of `Boxes` instead of lines.  This is the only place that you'll have nest loops for column and row.  It'll look something like this:

``` java
void populateBoard(int maxColumns, int maxRows) {
  for (int column = 0; column < maxColumns; column++) {
    for (int row = 0; row < maxRows; row++) {
      GameLine topLine = findOrCreateGameLine(LineType.HORZ, column, row);
      GameLine bottomLine = findOrCreateGameLine(LineType.HORZ, column, row+1);
      GameLine leftLine = findOrCreateGameLine(LineType.VERT, column, row);
      GameLine rightLine = findOrCreateGameLine(LineType.VERT, column+1, row);
      boxs.add(new Box(column, row, topLine, bottomLine, leftLine, rightLine));
    }
  }
}
```
Obviously, the constructor for `Box` will have to initalize a list of GameLines and populate it.  

## Playing the Game

The only other piece of data that you need is some place to hold the active player.  Let's call it `activePlayer`.

Now you can conceptualize the actual game play.  Create a method called `activateLine(LineType type, int column, int row)`.  This is what it does:

1.  Loop through `boxes` and count the number of completed `Boxes`, save this as `startingCompleted` (or something like that).
1.  Call `findOrCreateGameLine(type, column, row)` and set its `activated` field to true.
1.  Loop through the `Boxes` and do the following:
    1.  If the `Box` is completed, skip it.
    1.  Loop through its list of `GameLines`, and check to see if they are all activated.  If not, then go to the next `Box`.
    1.  Set the `Box` `completed` to `true`.
    1.  Ste the `Box` `owner` to the current player.
1.  Loop through `boxes` and count the number of completed `Boxes`.
1.  If the number of completed `Boxes` is greater than at the beginning, then stop.
1.  Flip the `activePlayer`

## Testing it

You can now write a testing method called `playGame()`.  Set up the starting stuff, call `populateBoard()` and then call `activateLine()` over and over simulating the moves.  All without a GUI.  At the end, you should be able to count up the owners of all of the boxes and get the right answer.  

You could run a simulation like the way this one runs:

![Simulation]({{page.simulation}})

If you can do that, **then** you can start to think about a GUI would interact with your gameplay.  The cool thing is that, if you do it right, almost none of the code you right to work without a GUI is going to get thrown away.

# Some Notes about Fields and Constructors

When you declare a field in a class you can specify a number of things:

1. The type of the data.
1. The visibility of the data.
1. The initial value of the data.
1. Whether or not the data can be changed.

So you can do something like this:

``` java
private String fred = "I am fred";
private final String george;
public final String MARVIN = "I am Marvin";
```
In this example, `fred` is just a regular variable, it's been initialized to "I am fred", but can be changed at any time.  `MARVIN` is essentially a constant.  It's `final`, so it can't be changed and it has the value "I am Marvin".

Now, `george` is a bit strange.  It's `final`, but it has no value!  What's going on here?

First, when I started out I never used `final` at all.  Why bother?  Then I've come to understand that it's really important.  When you're programming you really need to program exactly what you intend.  If something shouldn't be changed, then program it that way.  Don't rely on the fact that you know not to change it, or that there won't be any code that changes it.  If the nature of something is that it shouldn't be changed, if you **intend** to never change it, then code it that way with `final`.

Back to `george`.  Since `george` isn't initialized in the declaration, yet it's final.  This means that it **has** to be initialized in a constructor.

A constructor is the code that's run when `new` is performed on a class to *construct* an object.  You can have multiple constructors if there are multiple ways that you might want to create an object of a particular class.  Those constructors can only differ in the parameters that they take.

So:

``` java

public class Blah {

  private final String fred;

  public Blah() {
    fred = "I am fred";
  }

  public Blah(String fredsName) {
    fred = fredsName;
  }
}
```

Now let's think about `GameBox`.  It has the following fields:

1. int `column`
1. int `row`
1. List<GameLine> `sides`
1. boolean `completed`
1. BoxOwner `owner`

Of these, `column`, `row` and `sides` should all be set only when the `GameBox` is created.  Which means that they should all be `final`.  Clearly, we don't know what the values are when we write the class, so this means that they need to be set in the constructor, and they need to be passed to the construct from whatever code calls it.  This makes sense, no?

The fields, `completed` and `owner` both have initial values that we know right now (`false`, and `NONE`).  So they should be set in the declaration.  No need to deal with them in the constructor.

Let's think about `sides`.  It's a `List`, so the idea of `final` gets a little confusing.  Let's look at it this way:  For any given instance of `GameBox`, there is only ever going to be one `List` of `sides`.  You're never going to wipe it out and replace it with a new `List`.  So the list itself needs to be `final`.

But in Java, there's no immutable list (there is in Kotlin, which is really cool, but not Java).  That means that you'll always be able to add or remove elements of the `List`, no matter what you do.  So let's ignore that for now.

But as a container, you really should initialize it in the declaration, but we have to leave it empty until the constructor because we don't know what the sides are until the constructor is called.  So the declaration of `sides` should look like this:

``` java
  private List<GameLine> sides = new ArrayList<>();
```

Then when you get the sides passed to constructor you can do something like:

``` java
sides.addAll(top, bottom, left, right);
```

Finally, let's think about scope.  Generally speaking, fields should always be private in a class.  The idea is that the inner structure of a class is nobody else's business. This way, if you want to change the structure of a class, it can't impact any other part of your application because they can't reference it.

**This is important with what we are doing.  You'll see this when we go back to looking at the UI**

In order to allow other classes to read or update fields, we create methods called `getters` and `setters`.  Once you write a `getter` or a `setter`, you can't change its return type or its parameters because it's public and that would mean that you have to change other classes because you've broken them.  That's pretty much the point of `getters` and `setters`.  

Here's the neat thing:  You can't have a setter for a final field.  Because you can't update it.  So the whole concept ties together.  

The other thing is that you **don't just write getters for every field**.  You need to think about how the fields are going to be used, and who should be doing that work.  

Let's look at `column` and `row`.  Do any other classes need to know the `column` and `row` of a `GameBox`?

Potentially, we have the idea of looking for a particular `GameBox` by its coordinates.  Something like this:

``` java
gameBoxes.forEach(gameBox -> {
  if ((gameBox.getColumn() == column) && (gameBox.getRow() == row)) {
    return gameBox;
  }
});
```
But is that really the best way to do this?  Who should be responsible for deciding a `GameBox` is at a particular location?  Probably `GameBox` itself.  So what you should do, rather than create getters for column and row, is to create a method that says "yes" or "no" to a particular location:

``` java
public class GameBox {
  .
  .
  .
  public boolean isAtLocation(int column, int row) {
    return ((this.column == column) && (this.row == row));
  }
}

```
Now your for loop looks like this:

```   
  gameBoxes.forEach(gameBox -> {
    if (gameBox.isAtLocation(column,row) {
      return gameBox;
    }
  });

```

But hang on.  Do we ever need to find a `GameBox` by location for our game mechanics?  I can't think of any reason.  Do we use `column` and `row` anywhere inside `GameBox`?  I don't think so.

There's a really important concept called "YAGNI" (You Ain't Gonna Need It) in programming.  Don't write stuff that you think that you "might need" some time in the future.  Because, very often, you're not going to need it.  And if you do, you'll need it in different way than you thought.  And in the meantime, that stuff you don't need is sitting in your code and occupying space in your head every time you look at it.

So, let's get rid of `column` and `row` from `GameBox`.  Because YAGNI.  Just delete the field declarations, and don't pass them to the constructor.

Now the field list for GameBox looks like this:

1. List<GameLine> `sides`
1. boolean `completed`
1. BoxOwner `owner`

And the constructor call would be `new GameBox(top, bottom, left, right)`.  

One last thing to note:  Remember, our only game play action is `activateLine(type, column, row)`, which then triggers a whole bunch of logic.  The most important parts are locating a `GameBox` because one of its sides is the newly activated line, and then deciding if the `GameBox` has just become completed.  We'll use the same method as we did for row and column to find a GameBox - write a method `GameBox.hasLine(type, column, row)` and call it.

So no logic outside of GameBox needs to access the field `sides`.  So you don't need a `getter` for `sides` either.  And since the list is `final` you can't have a `setter` for it either.  

You can follow the same reasoning with GameLine.  You should find that you do need column and row as fields, because that's how we locate them in the cache, but you don't need `getters` or `setters` for them.  You'll need a `getter` and `setter` for `activated`.

See what you can do with that.
