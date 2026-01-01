---
title: "Temple of Giants"
layout: portfolio_post
date: 2020-05-01
---
## Build Pipeline
For this project, I set up automated builds using Jenkins. This build ran daily (after working hours), and we would be notified if there were any issues.

Shipping the game was also done through Jenkins, and a single click of a button was enough to trigger this process. Every single one of the builds we shipped went through this pipeline, which prevented people from accidentally adding (save) data or features that shouldn't be in the public build.

## Puzzle-Platforming Elements
<iframe class="column video-embed aspect-16-9 align-right" style="width: 360px" src="https://www.youtube.com/embed/NWt5s20SyWE" frameborder="0" allowfullscreen></iframe>
Another part of the project I worked on were the puzzle-platforming elements like moving- and falling platforms and pushable- and chained blocks. While working on these, I continuously reached out to the designers that would be working with these objects so that I could make sure that the behaviour of the object was still like what they envisioned it to be. Additionally, it also allowed me to get their input on how easy (or difficult) it was for them to place these in the level and adjust the objects to act the way they wanted them to in a given situation.


## Game Analytics
I also implemented a way to track analytics. Before this project, I hadn't worked with analytics, so I first looked into possible ways of tracking them. After doing this, the designers and I discussed what analytics we wanted to track for this game. I then adapted this to the research I did before, making sure that all of the analytics we wanted were trackable.

To track analytics, I used the [GameAnalytics plugin](https://github.com/GameAnalytics/GA-SDK-UNREAL) and wrapped its function calls in some helper function. For example, the time the player spent in a room is automatically calculated and recorded when either the `ExitRoom` or `EnterRoom` function is called.

Custom analytics that could be tracked:
- Level attempts, completes and fails
- Time per level, time per room
- Cause and location of death 
- The room in which players quit

## Character Inverse Kinematics
<iframe class="column video-embed aspect-16-9 align-right" style="width: 360px" src="https://www.youtube.com/embed/XF0EN7QcmnE" frameborder="0" allowfullscreen></iframe>
To add a level of polish to the game, I worked on using inverse kinematics for the placement of the character's feet when idle. This was my first time working with animation blueprints (and IK), so things did not go very smoothly at the start.<br>
One of the issues I ran into was that the character's legs were stretched when standing on an uneven surface, which was caused by the foot being pulled down rather than being pushed up. To fix this, I just changed the direction in which obstructions were found (up instead of down).<br>
Another issue was that a foot's target position constantly changed while standing still, resulting in the character tapping their foot impatiently. This was caused the position from which the target position was determined – a joint in the character's foot. This joint moved while adjusting the leg's pose, which sometimes caused the target position to keep changing between two positions. I solved this by simply limiting the timeframe in which the target position could change, but in hindsight, a better solution would have been to determine the target position from a fixed position.
