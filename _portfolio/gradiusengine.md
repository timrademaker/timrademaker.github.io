---
title: "GradiusEngine"
layout: portfolio_post
date: 2019-09-01
---
## Overall Engine Architecture
Before getting started on any implementation, I planned out the engine's overall layout in a UML diagram. I used "Game Engine Architecture" by Jason Gregory as reference for multiple features.<br>
In retrospect, I worked on the planning for too long before actually getting to any implementation. Part of the planning was based on speculation on what we might need, and not on hard requirements for the game (due to some communication issues between me and our gameplay programmer). I ended up overcomplicating some things that eventually went unused, so the fact that I should plan my code around project requirements better was definitely a takeaway for me.

## Editor Interface
For this engine, I worked on an editor that made it possible to edit entities and their properties in a scene. All component properties that have been registered can be modified through this.<br>
The editor also has the option to undo and redo actions, storing up to 20 steps in each direction.<br>
Aside from saving and loading levels, it is also possible to save and load specific entities as prefabs.

The editor can also be used to enter and exit play mode. Additionally, it is possible to pause the game's execution and edit the entities in their states at the time the game was paused.
