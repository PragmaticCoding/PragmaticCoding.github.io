---
excerpt: Looking at how clicks on the max hexes during setup can be used to remove units from the map.
permalink: /javafx/cybertanks/undeploy
sidebar_image: /assets/cybertanks/setup_sidebar.png
---

![CyberTank]({{page.banner}})setup_sidebar

# Introduction

As a user, it would be a big pain if, once you had deployed a unit on the hex map, you couldn't ever move it during setup.  It would probably be best to have a function that would allow a unit to be moved between hexes, but this seems like it would be complicated to do.  Since the goal is to get to a "bare-bones" working application first, we'll provide a function that will allow a unit to be removed from the map, and returned to the pool of units that can be deployed.  We'd have to provide this kind of functionality anyways, so doing this is going to save us some coding and still have fully functional game.

#
