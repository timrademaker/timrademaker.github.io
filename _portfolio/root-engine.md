---
title: "ROOT Engine"
layout: portfolio_post
date: 2020-02-01
---
## Blender Plugin for Level Editing
<iframe class="column video-embed aspect-16-9 align-right" style="width: 360px" src="https://www.youtube.com/embed/EMM5jI_h1hE" frameborder="0" allowfullscreen></iframe>
We knew that we would need some form of level editor for out engine, but didn’t think we would have the time to make our own (in-engine) level editor. It was recommended to us that we use Blender to edit levels, then export these levels to a GLTF file, and load this file into the engine.

We didn’t want all models to be loaded as plain `GameObject`s, so we needed some way of identifying different types of objects. For this, I decided to attach some metadata to objects to make identifying their type possible.<br>
Of course, I didn’t want people editing a level to have to know (and type out) the names of all available object types. To deal with this, I worked on [a plugin for Blender](https://github.com/timrademaker/ROOTEngine-BlenderPlugin), which adds a drop-down to all objects and allows the user to set the object type like that. The object names are read from a text file that the engine generates, so the user only has to set the path to this file once. It’s also possible to set a camera as the “main” camera, i.e. the camera that the renderer will use by default.

## Scrum Master
During this project, I also took on the role of scrum master. I had used scrum before this project, but didn’t know a lot about it. This role is definitely something that lies outside of my comfort zone, but I learned a lot from it.

## Automated Builds
For this project, I also set up automated builds using a freestyle job on Jenkins. This job used some batch scripts that were stored in the Perforce depot, and built the project, ran unit tests, and archived the build artifacts.

## Project Setup
The project setup was done using a Visual Studio solution. This solution held multiple projects: Game, Engine, Windows and Orbis, where Engine held platform-agnostic code and interfaces that were implemented in the platform-specific projects.<br>
For settings shared among multiple projects, I used project pages.

Before I set up the project, I first planned out the overall engine architecture (based on our requirements) in a UML class diagram. This made it easier to determine what setup would fit the project best.

## Multiplatform Support
One of the requirements for this project was that it would support two platforms (Windows and PS4). This was my first multi-platform project, so I looked at existing engines for ideas on how to set this up. The research I did here resulted in me writing [my first blog post]({% post_url 2020-04-04-setting-up-a-project-for-multi-platform-development-visual-studio %}).<br>
In the end, I chose to use interfaces for platform-dependent systems (like the renderer). The interfaces weren’t purely interfaces, as they had a static function that allowed users to create an instance of the system for the correct platform (without having to worry about which platform that was themselves).

The requirement of the project supporting PS4 was dropped partway through the project (as a result of possible measures against Covid-19 causing uncertainty on our ability to access the devkits). However, I did implement and test one feature for both Windows and PS4 before this: File IO using a `FileManager` class.<br>
While the PS4 implementation of this class ended up being unused, I still learned some things about multi-platform- and PS4 development by working on this.

## GLTF Model Loading Support
I worked on GLTF model loading support for this project (using [tinygltf](https://github.com/syoyo/tinygltf)), as we would need to load both meshes for the game’s visuals and an entire scene for the gameplay to take place in (as we used Blender as our level editor).<br>
This was my first time working with the GLTF file format, so I initially had some issues getting this to work. However, as other project teams also had to load GLTF models (due to this being a project requirement), I talked to some others working on GLTF model loading about issues I had or solved, which helped us all in getting the model loading implemented faster.
