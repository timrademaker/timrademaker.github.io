---
title: "Mazeum"
layout: default
date: 2020-12-01
---
{% assign project_slug = page.title | slugify %}
{% assign project_data = site.data.portfolio[project_slug] %}

<div class="columns">
    {% if project_data.image_path %}
    <div class="column">
        <img src="{{ project_data.image_path }}">
    </div>
    {% endif %}
    <div class="column">
    Mazeum was my entry for Search for a Star's Rising Star 2021 programming challenge, <a href="https://gradsingames.com/search-for-a-star/sfas-2021-the-finalists/">where I reached the finals</a>.
    <br><br>

    {{ project_data.description }}
    </div>
</div>

<div class="spacer" style="height:30px"></div>

## Procedural Generation
<img class="align-right" style="width: 360px" src="/assets/portfolio/mazeum/mazeum-procedural-generation.gif">
One of my goals for this project was to have procedurally generated levels.<br>
Rather than setting up two stages for the procedural generation (generation and verification), I decided to go with a one-stage approach to save development time. I made this possible by having the procedural generation work with a ruleset that prevented the verification from being needed. While this kind of procedural generation likely has its limitations, I did not run into any problems with it for this project.

## Research
For this project, I researched two reference games (Dreadhalls and Hitman 2) for their procedural generation and stealth mechanics, and looked into real-life museum structure to get inspiration for the layout of the museums in my game.<br>
My research on museum structure led to me to the conclusion that there are three different types of general museum layouts (though this is an oversimplification):

1. **Inter-connected rooms**: One room leads to another, with no hallways between rooms.
2. **Separate rooms**: Hallways connect rooms, and rooms have no direct connection to each other. The rooms fit around the hallway, not the other way around.
3. **Hybrids of the previous two types**: Rooms can be accessed through hallways, but there are also doorways directly leading from one room to another.

Based on this research, I concluded that separate rooms connected by a hallway would work best for my game. Closing off individual rooms is easy, and can force players to find another way into the room (through vents), potentially making the museum feel maze-like.

The research (along with the rest of my documentation) can be found on [the game's itch.io page](https://timrademaker.itch.io/mazeum).

## Reflection
If I were to work on a similar project again, I would go for in-editor procedural generation rather than runtime procedural generation. This would have allowed me to make the levels more enjoyable and diverse (even with a limited number of room templates), and it would have allowed me to do optimisation on the levels, for example by setting the lighting and most actors to static (rather than movable).<br>
Something that I *would* do the same is using [a Trello board](https://trello.com/b/KjMpOa7P/sfas-2021) (or something similar) to track my work. The board gave me a good overview of what I still had to do, and gave me some (short-term) project goals to work towards.

<h2>Additional Information</h2>
<div>
{% for detail in project_data.details -%}
    <b>{{ detail.title }}</b>: {{ detail.value }}<br>
{% endfor %}
{% if project_data.available_on -%}
    <b>Available on</b>:
    {% for platform in project_data.available_on %}
    <a href="{{ platform.link }}">{{ platform.name }}</a>{% unless forloop.last %},{% endunless -%}
    {% endfor %}
{% endif %}
</div>