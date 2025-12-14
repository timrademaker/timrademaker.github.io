---
layout: default
title: Project Archive
---
<h1 class="page-title">Project Archive</h1>

{% assign projects = site.portfolio | reverse %}
<div class="columns project-archive">
    {%- for project in projects -%}
        {%- include portfolio/project_tile.html project_title=project.title -%}
    {% endfor %}
</div>
