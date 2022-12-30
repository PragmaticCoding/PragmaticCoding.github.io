---
excerpt: "CyberTank Movement - Part 2"
permalink: /javafx/cybertanks/ct_move_2
---

# Moving the CyberTank by Turns

Now that we have the basic movement of the CyberTank sorted out, it's time to look at moving the CyberTank in the context of a turn.  Most importantly, this means that we can only move so many hexes at a time.

# Treads

CyberTank movement is determined by the condition of its treads.  As its treads become damaged, it's movement allowance gets smaller and smaller, until it cannot move at all.  

We're going to delegate all of this management to class called `Treads` and then add it to our CyberTank class as a property.

``` kotlin
class Treads {
   fun getMovesAllowed() = 4;
}
```
Clearly, this is just trivial now, but it will get more complicated as we go on.


# CyberTank

``` kotlin
fun pathForTurn(): List<Location> = mutableListOf(currentLocation).apply {
   for (idx in 1..treads.getMovesAllowed()) {
      if (move()) {
         this += currentLocation
      }
   }
}

private fun move(): Boolean = pathFinder.pathTo(currentLocation, determineDestination()).run {
   if (this.size > 1) {
      currentLocation = currentLocation.goDirection(movementRandomizer.randomize(currentLocation.directionTo(this[1])))
      return true
   }
   return false
}
```

# An Issue

One thing that became apparent from testing was that occasionally the CyberTank would bounce back and forth between two hexes as it alternately randomized clockwise and counterclockwise, or the reverse.  This is explicitly not allowed in the MOOSE rules, as it says that it can only dodge towards one direction each move.  

How to fix this?

The answer has to be in `MovementRandomizer`, but how to keep track of the turns?  The easiest way to do this is to instantiate a new `MovementRandomizer` for each turn, so it starts fresh for each turn.  In this way, `MovementRandomizer` doesn't need to know anything about turns, but will give the correct behaviour.  

The change to `MovementRandomizer` is fairly simple:

``` kotlin
class MovementRandomizer {

   private var counterClockWiseUsed = false;
   private var clockWiseUsed = false;
   fun randomize(direction: Direction): Direction {
      val random = Random.nextInt(0, 100)
      if (random < 60) return direction;
      if (random < 80) {
         if (!counterClockWiseUsed) {
            clockWiseUsed = true
            return direction.clockWise()
         } else {
            return direction
         }
      }
      if (!clockWiseUsed) {
         counterClockWiseUsed = true
         return direction.counterClockWise()
      }
      return direction
   }
}
```
Now any instance of `MovementRandomizer` will possibly randomize to only the left or the right during its existence.  

We implement this, by removing `movementRandomizer` as a field, and instantiating it in `CyberTank.pathForTurn()` and passing it to `CyberTank.move()`.

``` kotlin
fun pathForTurn(): List<Location> = mutableListOf(currentLocation).apply {
    val movementRandomizer = MovementRandomizer()
    for (idx in 1..treads.getMovesAllowed()) {
       if (move(movementRandomizer)) {
          this += currentLocation
       }
    }
 }

 private fun move(movementRandomizer: MovementRandomizer): Boolean = pathFinder.pathTo(currentLocation, determineDestination()).run {
    if (this.size > 1) {
       currentLocation = currentLocation.goDirection(movementRandomizer.randomize(currentLocation.directionTo(this[1])))
       return true
    }
    return false
 }
```

# Integrating This With the GamePlay Logic

Now that we have the CyberTank moving in a turn-by-turn manner, we need to integrate this back into game logic, as we still want to animate the CyberTank movement on the hex map.  

We fundamentally altered the way that this works.  It was still necessary that the path be recalculated each hex, because of the possible changing targets and the randomization, but CyberTank is probably only going to care about the end result.  Since the map needs the path taken for animation, `CyberTank.pathForTurn()` will return the entire path.

This actually takes us back the original design for very early testing.  The movement plotting was done in the background of a `Task` and then the results sent back to the Interactor for animation.  Now it looks like this:

``` kotlin
private val cyberTank = CyberTank(unitLocations)

fun animateTank(cyberPath: List<Location>) {
   PauseTransition(Duration(1000.0)).apply {
      var movesTaken = 0
      setOnFinished {
         if (movesTaken <= (cyberPath.size - 2)) {
            handleUnitMovement(UnitMovement(cyberPath[movesTaken], cyberPath[movesTaken + 1], UnitType.CT1))
            movesTaken++
            play()
         }
      }
      play()
   }
}

fun calculateTankPath(): List<Location> = cyberTank.pathForTurn()
```
We've promoted the CyberTank variable to a field, which seems to make sense.  `GameInteractor.calculateTankPath()` now simply delegates to `CyberTank.pathForTurn()` and return the `List<Location>`.

The animation code has been cleaned up and made more "Kotlin-like".  All of the action takes place inside the `apply{}` scope function.  The `moveTank()` method was redundant since it just contained the same functionality as `handleUnitMovement()`, so it's been removed and replaced with a call to `handleUnitMovement()`.
