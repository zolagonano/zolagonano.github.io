---
layout: homepage
title: Projects
description: List of the projects that I'm currently working on and maintaining.
---

## Projects

 I like working on free and open-source software and creating free content. Check out the list below for some of the projects I'm currently working on and maintaining:

{% for project in site.data.projects %}- [{{ project.name }}]({{ project.url}} ): {{ project.description }}
{% endfor %}
