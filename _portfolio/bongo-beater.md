---
title: "Bongo Beater"
layout: portfolio_post
date: 2020-01-01
---
## Procedural Generation
For the procedural generation based on music, I first researched if and how other games did this. I ended up using onset detection for this, which was possible by using the [Audio Synesthesia plugin](https://docs.unrealengine.com/en-US/Engine/Audio/Synesthesia/index.html). This wasn't without problems: The onset detection wasn't available outside of the editor.<br>
I dealt with this by exporting the generated data from the editor to a csv file, and importing this file as data table, which could then be used by the game.

## Challenges
For this project, we had to create a prototype from scratch in only three weeks. We dealt with this by splitting up the work as soon as we could, and by asking our teammates for help as soon as we ran into issues we couldn't quickly deal with ourselves.

This was the first VR game I worked on. The rest of the team had no experience with working on VR games either, so they couldn't help me with setting up player controls. Instead, I looked at Unreal Engine's documentation regarding VR. Looking at the example VR project that came with Unreal Engine also proved to be a good idea, as I was able to adapt the implementation of VR controls there to our own project.
