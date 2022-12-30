---
excerpt: Intermission - Progress So Far"
permalink: /javafx/cybertanks/intermission_1
---

# What Have We Achieved So Far?

I thought this would be a good place to take a pause and look at how this project is going.  To me, it feels like we've come a long way.  We have a map loaded from a database, we've given the user the ability to deploy his forces on the map so that the game can start, and we have a CyberTank moving around the map.  The CyberTank already interacts with the defending forces that have been deployed, and alters its path to respond to the placement of those forces.



# The Code Structure

My code analyzer tells me that we have 990 lines of code over 47 classes for the whole project.  That count includes the import statements, and in the largest classes this is over 20 lines of code.  I'm guessing that we can deduct about 300 lines over the 47 classes to leave about 700 lines of "real executable" code.  

Included in this are a few classes that we haven't looked at yet, the ones that display the schematic of the CyberTank.  I wrote this very early on when I was messing with some UI look and feel stuff, and I've just left it there as it isn't getting in the way.  But this is another 80 or so lines of code that shouldn't count to our code total.  So let's call it about 600 lines of working code to get to where we are now.

The largest class has about 55 lines of real code, and the next largest has about 50.   The vast majority of classes have under 30 lines of executable code.  

Counting lines of code doesn't tell us a lot about code quality, but it does say something about how we've handled complexity and kept scope issues in check by applying YAGNI.  Unless you work at it, it's fairly difficult to cram something really complicated into a 10 line class.

I feel like the code so far has maintained a good amount of separation of duties.  Classes are directly responsible for only one thing, with other classes created to delegate sub-tasks. Perhaps the biggest example of this is `Tread`, which only has one method, and even that just returns a hard-coded value.  Maybe, at this point, it could be argued that it's a violation of YAGNI, but it's still just one line of code and doesn't represent a significant investment in complexity.  

# Problems to Come

I can see a few issues down the road.  One thing that's causing me some cognitive dissonance is the `UnitLocations` structure and how it carries information that duplicates that held in `TileModel`.  You can see that in `GameInteractor.handleUnitMovement()`.  This method updates the `TileModels` and then updates `UnitLocations`.  If you look closely, though, you'll see that the `TileModel` has an additional check to make sure that the move is valid, but there's no similar check on `UnitLocations` updates.  So it is possible that the two can get out of sync.  

Talking about `TileModel`; this is causing me some stress.  There's nothing wrong with having critical data stored only in a Presentation Model - we see this all the time with regular CRUD type form layouts - but `TileModel` is feeling a bit too promiscuous at this point.  It's used all over the place, and I think this is starting to create a lot of coupling.  

Looking ahead, I think this is going to be an issue when we look at animating missile paths and explosions.  My expectation is that we'll end up taking `TileController` and renaming it to simple `Tile`.  Then we'll add some public methods to allow access to the `TileModel` data, and to trigger actions and animations.  Then we'll have to refactor the application to keep track of `Tile` instead of `TileModel`.  

# Future Complexity

The big issue that's been kicked down the road for now is the impact of Terrain types on movement and accessibility.  My feeling right now is that 99% of this complexity is going to fall into `PathFinder` and `MovementRandomizer`, particularly the former.  `PathFinder` right now finds a shortest path to the destination by always moving directly towards the destination.  However, when hexes have different movement costs, that direct path might no longer be the fastest path.  

Currently, there's nothing special about any particular unit of any given type in a location.  If there's a Heavy Tank in hex [7,3] then it's just any Heavy Tank.  So the `UnitLocations` structure of just having a list of `Locations` for each `UnitType` is fine.  But there's going to become some differentiations once we start looking at combat results.  Units can become "disabled", and therefore different from other units of the same type.  Also, once we look at gameplay for the human player, we'll have the situation where some units of a given type have moved in a turn, while others have not.  
