---
title: "NIS Classics Vol. 2"
layout: portfolio_post
date: 2021-09-01
---
## The project
For this project, I assisted in porting two classic games to modern platforms (PC and Nintendo Switch). As this was a porting project, we worked with a large codebase that was unfamiliar to the entire team.<br>
During the project, most developers became more familiar with one game's codebase than with the other, and ended up with an area of "expertise" in this project. Personally, I did more work on Makai Kingdom and ended up learning a lot about the game's custom scripting language, how these scripts were handled in the game, and how the asset pipeline worked.

## Mouse Support
<div class="columns">
    <div class="column">
        <p>One of the new features of this port would be the addition of mouse support for the PC version of the game. I worked on this for Makai Kingdom, and set up some mouse support functions that were usable throughout the whole project with minimal changes to existing code. This was possible by, for example, adding code to create a button to the text drawing function that was already being used throughout the project.</p>
    </div>
    <iframe class="column video-embed aspect-16-9" src="http://www.youtube.com/embed/p8ZuvkX4NAw" frameborder="0" allowfullscreen></iframe>
</div>

## Map Rendering Optimization
Makai Kingdom did not reach our target framerate of 60 fps on the Nintendo Switch and lower-end PCs. After profiling, I found that the biggest bottleneck was the map rendering. The maps consist of fixed-size rectangles that were rendered using a draw call each, so performance would deteriorate as the game's dungeons grew in size. To improve the framerate, I batched squares that used the same texture together so that they could be drawn in a single draw call. This resulted in the game running at 60 fps on the Switch, regardless of the size of the map.

## Game Data Load Speed-Up
Starting up Makai Kingdom on the Nintendo Switch took quite some time, which I improved by reducing the amount of data that was loaded on boot.

## Debug Features
For Makai Kingdom, I added various new debug features to assist the testers. Specifically, I added an option to order the "random" names (and added control over the parameters that controlled these), made it possible to unlock all character classes, and made it possible to buy all items from the game in the shop.<br>
As some debug features required more than a single keybind to be used properly, I added an ImGui-window in order to make these features easier to use.

## Achievements
We also added achievements to the PC version of the games. Adding these was an interesting challenge, as finding out where in the code some achievements had to be unlocked was not always easy. To make unlocking some achievements possible, I even ended up extending Makai Kingdom's scripting language.

## Bug Fixing
Due to the nature of this project, a lot of the work consisted of fixing bugs that were caused by porting due to differences between the old and new target platforms. Fixing some of them proved to be tricky, as they relied on the way different systems work together. As I wasn't familiar with the full codebase of both games, the bugs sometimes required a bit of speculation to track down when debugging didn't quite cut it.<br>
Aside from bugs introduced by the porting process, I also fixed some legacy bugs at the request of the publisher.
