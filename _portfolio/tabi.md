---
title: "Tabi"
layout: portfolio_post
date: 2020-05-02
---
## Renderer
Tabi's renderer is implemented using OpenGL, but the renderer's interface is based on D3D12 (with some differences with OpenGL taken into account). Originally, the interface was designed around OpenGL, but this was not very compatible with other graphics APIs (like D3D11/12, Vulkan, and console APIs). The interface is designed around two classes: the device and the command list. The former is mainly responsible for resource creation and destruction, whereas the latter handles things like binding buffers and textures and drawing.<br>
I also integrated [imgui](https://github.com/ocornut/imgui) into the project using a custom backend that utilizes the renderer's abstraction in order to have imgui work out-of-the-box on any platform for which I might implement rendering in the future.

## Entity Component System
In a previous engine project, I also worked on an entity component system. However, that ECS was not data-oriented. Copy-pasting that implementation would have been easy enough, but I wouldn't have learned anything by doing that.

In my current implementation, an entity is simply an ID. Components are stored in `ComponentArray`s, and systems are managed by a `SystemManager`. Components are not required to inherit from any specific class, but all systems need to implement the `ISystem` interface.

## Event System
I also worked on an event system. This makes it possible to, for example, have objects that act based on input, but don't have to check if a specific key is down every update.<br>
Working on this resulted in me writing two articles on the topic: [One on an event system and possible considerations]({% post_url 2020-09-10-event-system-cpp %}), and [one on making the events consumable]({% post_url 2020-09-11-event-system-cpp-consumable-events %}) (which I chose not to do in this project).

## Collision Detection
Tabi's collision detection system uses the Gilbert–Johnson–Keerthi distance algorithm. The implementation is based on [this video by Casey Muratori](https://www.youtube.com/watch?v=Qupqu1xe7Io), though I used some other sources for it as well.<br>
In terms of functionality, it would have been better to integrate a third-party library like Bullet or PhysX, but I decided to implement GJK as a learning experience. I implemented collision detection using the Separating Axis Theorem before and considered doing the same here, but I decided against it after comparing the two. Of course, GJK being something I had not implemented before played a role in this decision as well, but the decision was largely based on other considerations like performance in 3D space.<br>
GJK does not provide info on penetration depth of a collision, so I plan to implement the Expanding Polytope Algorithm for that at some point.

## File IO
This project's file IO is similar to that of the [ROOT Engine]({% link _portfolio/root-engine.md %}), but has one major difference: The `FileManager` class no longer exists. Instead, `IFile` is used to open files. The reason for this change is that I found that there was no real point to a file manager while working on the ROOT Engine. On the contrary, sharing an opened file with another object could lead to unexpected behaviour when the file cursor is moved unexpectedly.

## Input System
The current input system is event-based and is split up between the input manager and input handler. The manager is platform-agnostic and handles the functionality that is shared between all platforms (e.g. notifying the user of a key going down), whereas the handler is an interface that the manager uses to poll input. Users only have to interact with the input manager, and new platforms only have to implement the input handler interface.<br>
The implementation on Windows uses [Gainput](https://github.com/jkuhlmann/gainput) internally, though I also implemented some things that the library does not handle (e.g. scroll wheel delta).

## Logger
The logger is a wrapper around [spdlog](https://github.com/gabime/spdlog). By default, there are two loggers: one for the engine and one for the game. Users can create new loggers as they see fit, and sinks can be added and removed based on project needs.<br>
Tabi's renderer, for example, has its own logger so that filtered logging levels can be set separately from the rest of the engine, and to make finding log messages from the renderer easier.

## GLTF Model Loader
The GLTF model loader in Tabi uses [tinygltf](https://github.com/syoyo/tinygltf) just like it did in [ROOT Engine]({% link _portfolio/root-engine.md %}), with some improvements over the implementation for that project.<br>
As parsing GLTF files and extracting data from them is not the fastest way of getting the model loaded in the game, I intend to move the GLTF parsing out of the engine into a "proper" asset pipeline at some point. For this, I will probably use [another personal project I worked on](https://github.com/timrademaker/hako).

## Resource Manager
To prevent the same resource from being loaded into memory multiple times if accessed from different parts of the code, I added a resource manager to Tabi. This manager keeps track of which assets are loaded, and unloads them after they are no longer in use. Unloading assets directly after they are no longer used might not be the best way of doing things, as some resources might be needed again just a bit later. This is something that I will have to take another look at in the future.

## CMake
For the project's setup, I chose to use CMake. I didn't have any prior experience setting up a project with it and initially had some trouble getting things set up the way I wanted to, but I pushed on and managed to get it to work.
