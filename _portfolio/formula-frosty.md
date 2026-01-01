---
title: "Formula Frosty"
layout: portfolio_post
date: 2019-05-01
---
## Adaptive Camera
<iframe class="column video-embed aspect-16-9 align-right" style="width: 360px" src="https://www.youtube.com/embed/OWc2_hskgd8" frameborder="0" allowfullscreen></iframe>
The camera we (initially) wanted for the game was one that zoomed in and out based on the distance between the players. While working on this, I closely communicated with a designer to get the camera to behave exactly how they wanted. To make it easy to adjust the camera's behaviour, I exposed some variables with a clear description on what they affected.

## Menus
<iframe class="column video-embed aspect-16-9 align-right" style="width: 360px" src="https://www.youtube.com/embed/Eg4JLcngdm0" frameborder="0" allowfullscreen></iframe>
Together with a designer, I worked on the main menu, pause menu and character selection. While the main- and pause menu were both widget-based, we wanted something more interesting for the character selection. To do this, we placed the vehicles on the racing track, and had the players' camera fly between the vehicles. To allow multiple players to select a vehicle at the same time, this was made split-screen.<br>
The menu being split-screen while the game was not was something that was seen as confusing, so we changed this to a single-camera vehicle selection. Luckily, we were able to re-use the implementation that was used for the split-screen vehicle selection (with some small modifications) as the behaviour was largely the same.

## Player Ranking
To determine the ranking of all the players (though most importantly, who was in first place), I used Unreal Engine's splines. By converting the world position of the player to a position on the spline, it was possible to determine how far along the spline the player was located. With this information, it was trivial to determine player rankings.<br>
As the track was generated around a spline, it didn't take any effort to update the spline used for ranking when the track was changed (because this was the same spline as the track spline).

## Challenges
This was my first time working with Unreal Engine. By looking at Unreal Engine's documentation and example projects, and asking teammates for help if needed, I learned what I needed to know as I went along.
