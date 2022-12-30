---
excerpt: "CyberTank Movement - Part 1"
permalink: /javafx/cybertanks/ct_move_1
sidebar_image: /assets/cybertanks/setup_sidebar.png
---

# Moving the CyberTank

The first thing we're going to tackle in gameplay is the CyberTank movement.  

The rules from MOOSE or fairly simple, the CyberTank will move towards the Command Post unless there is a Howitzer within 8 hexes of the CyberTank.  8 hexes is the range of a Howitzer, so this just represents that the CyberTank would prefer to rush the Howitzer and blow it up, rather than be pummelled by it as it proceeds towards the Command Post.

There is also a table in MOOSE to use two dice to somewhat randomize the movement of the CyberTank.  We don't need to replicate dice throws, but we can create a class to use a random number between 0-100 and adjust some ranges to impact the movement of the CyberTank.

# Hex Math

Hex maps are a bit different from square grids, and some of the handling can get a bit obtuse.  So we'll put all of that stuff into a set of classes that can just deal with it, then our AI routines and rule checking code won't need to deal with it at all.

One thing that we're not going to deal with at this point is terrain.  There are rules about hexes that are impassable, or cost more to traverse based on the terrain in them.  All that stuff will get incorporated later on.  For now, we just want to get the CyberTank moving according to some basic rules.

## Directions

When we talk about moving the CyberTank, we need to talk about which way it's facing, and which direction it's going to move.  So we'll introduce a new class to hold this.

``` kotlin
enum class Direction {
   N, NE, SE, S, SW, NW;

   fun clockWise() = when (this) {
      N -> NE
      NE -> SE
      SE -> S
      S -> SW
      SW -> NW
      NW -> N
   }

   fun counterClockWise() = when (this) {
      N -> NW
      NE -> N
      SE -> NE
      S -> SE
      SW -> S
      NW -> SW
   }
}
```
You can see that we have six directions (surprise!), but no East or West.

## Location

There's a bit more to this to integrate with Directions and to calculate the Direction from one location to another.

``` kotlin
data class Location(val column: Int, val row: Int) {
   fun equals(otherLocation: Location?): Boolean = otherLocation?.let {
      (this.row == otherLocation.row) && (this.column == otherLocation.column)
   } ?: false

   override fun toString(): String {
      return "$column:$row"
   }

   fun goNorth(): Location = Location(column, row - 1)

   fun goSouth(): Location = Location(column, row + 1)

   fun goNorthEast(): Location = Location(column + 1, if (column.isOdd()) row - 1 else row)

   fun goSouthEast(): Location = Location(column + 1, if (column.isOdd()) row else row + 1)

   fun goNorthWest(): Location = Location(column - 1, if (column.isOdd()) row - 1 else row)

   fun goSouthWest(): Location = Location(column - 1, if (column.isOdd()) row else row + 1)

   fun directionTo(otherLocation: Location): Direction {
      if (column == otherLocation.column) {
         return if (row >= otherLocation.row) Direction.N else Direction.S
      }
      return if (otherLocation.column > column) {
         if (column.isOdd()) {
            if (otherLocation.row < row) Direction.NE else Direction.SE
         } else {
            if (otherLocation.row <= row) Direction.NE else Direction.SE
         }
      } else {
         if (column.isOdd()) {
            if (otherLocation.row < row) Direction.NW else Direction.SW
         } else {
            if (otherLocation.row <= row) Direction.NW else Direction.SW
         }
      }
   }

   fun goDirection(direction: Direction) = when (direction) {
      Direction.N -> goNorth()
      Direction.NE -> goNorthEast()
      Direction.SE -> goSouthEast()
      Direction.S -> goSouth()
      Direction.SW -> goSouthWest()
      Direction.NW -> goNorthWest()
   }

}

fun Int.isOdd(): Boolean = (this % 2 != 0)
```

Remember that with hex maps the rows are zig-zag, so the row numbers of the squares that aren't directly north or south depend on whether or not the location is in an odd or even column.  This calculates it.

## PathFinder

One of the oddities about hex maps is that the easiest way to determine distance is to just map out a direct path from source to target and count up the steps.  It might be possible to do this with some kind of math, comparing the row and column of the two hexes, but I've never seen this done.  Probably because I usually encounter hexes in tabletop gaming, and you just count them.  So that's what we'll do here.  We're going to need the paths anyways.

``` kotlin
class PathFinder() {

   public fun pathTo(source: Location, destination: Location): List<Location> {
      val results = mutableListOf<Location>(source)
      println(source)
      if (source.column == destination.column) {
         if (source.row == destination.row) {
            return results
         }
         results += pathTo(if (source.row > destination.row) source.goNorth() else source.goSouth(), destination)
         return results;
      }
      val nextColumn = if (destination.column > source.column) source.column + 1 else source.column - 1
      results += pathTo(if (source.column.isOdd()) {
         Location(nextColumn, if (destination.row < source.row) source.row - 1 else source.row)
      } else {
         Location(nextColumn, if (destination.row <= source.row) source.row else source.row + 1)
      }, destination)
      return results
   }

   public fun rangeTo(source: Location, destination: Location): Int = pathTo(source, destination).size
}
```

Notice that `pathTo()` is a recursive function.  The basic operation is to determine the next step in the path, then call `pathTo()` from that location and continue on until the source and the destination locations are the same.  Then it rolls back up to the original source.  

# The CyberTank AI

Originally, there was just going to be a `CyberTankBrain` class, and maybe there still will, but with the inclusion of `currentLocation` it starts to be more general than just the AI.  Also, in the next step, we're going to need to add in something about the treads, as this controls how far the CyberTank can move in a single turn.  So, we'll just call this class `CyberTank` and we won't need to change it later...

``` kotlin
class CyberTank(private val unitLocations: UnitLocations) {

   private val pathFinder = PathFinder()
   var currentLocation = Location(10, 15)
   private val movementRandomizer = MovementRandomizer()

   fun move(): Boolean {
      val path = pathFinder.pathTo(currentLocation, determineDestination())
      if (path.size > 1) {
         currentLocation = currentLocation.goDirection(movementRandomizer.randomize(currentLocation.directionTo(path[1])))
         return true
      }
      return false
   }

   fun determineDestination(): Location = findClosestUnitInRange(UnitType.HWZR) ?: findClosestUnitInRange(UnitType.CPA) ?: Location(1, 1)

   private fun findClosestUnitInRange(unitType: UnitType): Location? {
      println("Looking For: $unitType")
      val units: List<Location> = unitLocations.getUnitLocations(unitType)
      if (units.isNotEmpty()) {
         val shortestRange: Int = units.minOf { location -> pathFinder.rangeTo(currentLocation, location) }
         if (shortestRange <= unitType.range) {
            return units.first { location -> pathFinder.rangeTo(currentLocation, location) == shortestRange }
         }
      }
      return null
   }
}
```

So far, we really have just two main functions; determine the target location to move to, and move one hex in the correct direction.  Note that the target can change during a turn if the CyberTank moves within range of a Howitzer during the turn.  in the event that there isn't a Command Post or Howitzer on the map, the CyberTank will just head to Location [1,1].

Also, `currentLocation` isn't private, and it's the responsibility of the CyberTank to keep track of it's own location.  The `move()` method returns true or false depending on whether or not the CyberTank did change it's location during `move()`.

We need to look at the movement randomizer.  Currently, it's very simple:

``` kotlin
class MovementRandomizer {

   fun randomize(direction: Direction): Direction {
      val random = Random.nextInt(0, 100)
      if (random < 60) return direction;
      if (random < 80) return direction.clockWise()
      return direction.counterClockWise()
   }
}
```

Eventually, we'll need to deal with terrain issues, and some rules that prevent the CyberTank from going left and right in the same turn.  But for now, this gets us started.

# Running the AI

At this point, we have something that can actually run and move the CyberTank, and we just need to integrate it with the main gameplay logic to see how it works.  This is fairly easy.  There was a `Button` included at the bottom of the original screen to run some early testing, and this can just be used to move the CyberTank until it reaches its destination and `CyberTank.move()` returns false.  Let's look at the logic in the main Interactor:

``` kotlin
fun animateTank(locations1: List<Location>) {
    val cyberTank = CyberTank(unitLocations)
    val pause = PauseTransition(Duration(1000.0))
    pause.setOnFinished {
       val didMove = moveTank(cyberTank)
       if (didMove) {
          pause.play()
       }
    }
    pause.play()
 }

 private fun moveTank(cyberTank: CyberTank): Boolean {
    model.consumeTile(cyberTank.currentLocation) { tileModel ->
       tileModel.occupierProperty.set(UnitType.NONE)
       true
    }
    val didMove = cyberTank.move()
    model.consumeTile(cyberTank.currentLocation) { tileModel ->
       tileModel.occupierProperty.set(UnitType.CT1)
       true
    }
    return didMove
 }
```

We can ignore the `locations1` parameter in `animateTank()`, as it was originally used for the test animation and it was easier to leave the call structure and original `Task` in place.  Now, `animateTank()` handles the setup of the animation for the CyberTank movement.  Essentially, it's a `PauseTransition` with the CyberTank movement in the `OnFinished EventHandler`.  If the CyberTank does move, then the `PauseTransition` runs again.

The method, `moveTank()` removes the CyberTank from the hex map at it's current location, then calls `CyberTank.move()` and adds the CyberTank back to the map at its (possibly) new current location.

Now, you can put a Command Post and/or Howitzers on the map, click the `Button` and the CyberTank will appear at the bottom of the map and then start moving around until it gets to it's destination.

That's all for a start.  We now have a CyberTank that will move around the map!
