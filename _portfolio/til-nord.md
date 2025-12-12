---
title: "Til Nord"
layout: portfolio_post
---
## Analytics
### Heatmaps
Something I worked on for this project is a script to generate heatmaps for player location, speed, and the location where the player bumped into something. This script is written in Python, using the numpy and Matplotlib libraries.

<div class="columns">
    <div class="column">
        {% include image.html url="/assets/portfolio/til-nord/location-heatmap.png" description="Heatmap showing player location" %}
    </div>
    <div class="column">
        {% include image.html url="/assets/portfolio/til-nord/bump-location-heatmap.png" description="Heatmap showing where players bump into objects" %}
    </div>
</div>

The heatmap data is collected through a component attached to the player, which records the player location and velocity at set intervals. The data is regularly flushed to a Spreadsheet, [using a plugin created by a teammate](https://github.com/jenspetter/SpreadsheetsWithUnreal).

I also worked on a WPF application that functioned as a visual interface for the heatmap generation script ([available on GitHub](https://github.com/timrademaker/Til-Nord-Heatmap)), which prevents users from having to download the heatmap data manually, and makes it so that they don't have to remember the command-line flags for the heatmap generation script if they want to change the configuration.

<div class="columns">
    <div class="column">
        {% include image.html url="/assets/portfolio/til-nord/heatmap-tool-main-screen.png" description="Heatmap tool main screen" %}
    </div>
    <div class="column">
        {% include image.html url="/assets/portfolio/til-nord/heatmap-tool-settings-screen.png" description="Heatmap tool settings screen" %}
    </div>
</div>

A new Spreadsheet tab is automatically created for every game build to allow for the versioning of location data, making it possible to see the effect of certain changes.

The script and application were used by designers so they could figure out which areas were under- or overused, and artists used it to find out which areas were more important when set dressing.

## Build Pipeline
For this project, I set up a build pipeline using Jenkins. To allow a more complex pipeline that would still be maintainable, I worked on a Jenkins Shared Library ([available on GitHub](https://github.com/timrademaker/JenkinsPipelineUtilities)). This library made it possible to, among other things, build, package and run tests on Unreal Engine projects, and to ship a build to Steam.

The build pipeline I set up was based on research I did on build pipelines in the games- and software industry, where I looked at build pipelines at Rare and IBM, among others.

{% include image.html url="/assets/portfolio/til-nord/build-pipeline.png" description="Build Pipeline used to deploy the build to Steam" %}

## Submit Pipeline
For this project, I also set up a submit pipeline that developers had to go through in order to submit their changes to the project. Before files could be submitted (for review), some pre-submit checks were run.<br>
The submit pipeline, like the build pipeline, was based on research.

Because we shared the build server with other teams, there were some limitations to what we could do with it. For example, running the build pipeline (or part of it) for every submit or review was out of the question. As such, I had to to find a different way of making the submit pipeline prevent issues from reaching the depot.

{% include image.html url="/assets/portfolio/til-nord/submit-pipeline.png" description="Submit Pipeline" %}

### Submit Tool
To automate the pre-submit checks, I worked on a submit tool. Instead of submitting files using the regular "Submit"-button in Perforce, developers would run this submit tool on their pending changelist. If all checks passed, they would be asked to confirm that they wanted to submit the changelist (for review), and if it failed, they were told at what stage the check found issues, and what those issues were.

{% include image.html url="/assets/portfolio/til-nord/pre-submit-check.png" description="Pre-Submit Check" %}

#### Automated Tests
One of the stages of the pre-submit check was the automated testing phase. As our team had folder structure conventions that made it possible to figure out which feature was being changed, the pre-submit check was able to run tests covering this feature. This was not perfect, as features that other features relied on being changed did not necessarily trigger all covering tests.<br>
To make sure that changes to code were properly tested, I worked on a tool that mapped code to the unit tests that cover the code, which goes further than just surface-level dependencies. This way, it was possible to only run tests that were relevant to the changes.

### Pre-Submit Reviews
Pre-Submit reviews were also part of the submit pipeline. These were not just for programmers to review other programmers, but also designers getting reviewed by programmers. Overall, this lead to more maintainable code and blueprints in the project, and occasionally prevented issues from reaching the depot and affecting other developers.

## Crash Reporter
The default Unreal Engine crash reporter requires a server to be set up for the crash reports, which I thought was a bit overkill for a project of this scale. While it wasn't strictly necessary to work on this, I thought it would be a good way to learn a bit more about C# and WPF.<br>
The crash reporter sends crash reports to a Discord webhook, and includes information about the version of the game.<br>
Sending crash reports to a channel on Discord is definitely not the best idea for a game with a lot of players, but I was not expecting a lot of crash reports when I worked on this.

The source code for the crash reporter [can be found on GitHub](https://github.com/timrademaker/UECrashReporter).

{% include image.html url="/assets/portfolio/til-nord/til-nord-crash-reporter.png" description="Custom Crash Reporter" %}

## Input Settings
Setting up input settings for this game was an interesting challenge, as I was tasked with creating input settings that would allow users to bind any button, trigger or stick to any game action or axis. For keyboard, this was mostly straight-forward, but adding gamepad input to the mix made this more difficult. For example, steering left and right with a gamepad is usually done with just a single stick, whereas these actions are represented by two keys on a keyboard.

I made this work by adding a variable to the input system where pairs of opposing axes (like steering left and right, or accelerating and braking) could be added. This list would then be used during rebinding for the following logic: When binding a stick to a "paired" bind, this bind needed to be applied to both left and right. However, if a button or trigger was bound to one of these axes, the other key has to be unbound if it's a stick.

In the end, it was decided that not all input settings should be rebindable, but working on this was interesting regardless.

## Saving and Loading
I set up saving and loading for various aspects of the game using Unreal Engine's built-in save system. This was mostly done using a per-feature save, to allow players to delete their game progress without deleting all of their settings (and to make save file classes easier to navigate during development).

## Quality Assurance
One of my responsibilities during this project was quality assurance, as the team's QA engineer and release manager.<br>
I organized QA testing sessions twice a week, where half of the team helped test the game. This helped us catch bugs that the submit pipeline didn't catch, and allowed us to verify that certain bugs had been fixed.

The testing was done on a separate branch on Steam. If the build was stable enough and the team leads and producer deemed the bugs that were still in the build acceptable, I made a release candidate build of the tested version. This version went through some more testing before the public branch was updated.
