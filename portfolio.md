---
layout: default_wide
title: Portfolio
---
<h1 class="page-title">Portfolio</h1>
<h2 class="block-heading">Game Projects</h2>

<div class="columns">
    {%- include portfolio/project_tile.html project_title="King of Meat" -%}
    {%- include portfolio/project_tile.html project_title="NIS Classics Vol. 2" -%}
    {%- include portfolio/project_tile.html project_title="Til Nord" -%}
</div>

<div class="spacer" style="height:30px"></div>

<h2 class="block-heading">Engine and Tools Projects</h2>
<div class="columns">
    {%- include portfolio/project_tile.html project_title="Tabi" -%}
    {%- include portfolio/project_tile.html project_title="ROOT Engine" -%}
    {%- include portfolio/project_tile.html project_title="GradiusEngine" -%}
</div>

<div class="spacer" style="height:30px"></div>

<a href="{% link _additional_pages/project_archive.md %}">Project Archive</a>
