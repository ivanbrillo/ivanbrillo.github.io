---
layout: page
title: Projects
permalink: /projects/
description: Research projects, open-source work and things I have built along the way.
nav: true
nav_order: 4
display_categories: [research, AI projects, fun]
category_titles:
  research: Research
  AI projects: AI & software projects
  fun: Open source & engineering
horizontal: false
---

<!-- pages/projects.md -->
<div class="projects">
{% if site.enable_project_categories and page.display_categories %}
  {% for category in page.display_categories %}
  {% assign categorized_projects = site.projects | where: "category", category %}
  {% assign sorted_projects = categorized_projects | sort: "importance" %}
  <div class="project-category" id="{{ category | slugify }}">
    <h2>{{ page.category_titles[category] | default: category }}</h2>
    <span>{{ sorted_projects | size }}</span>
  </div>
  <div class="projects-grid{% if forloop.first %} projects-grid-lg{% endif %}">
    {% for project in sorted_projects %}
      {% include projects.liquid %}
    {% endfor %}
  </div>
  {% endfor %}
{% else %}
  {% assign sorted_projects = site.projects | sort: "importance" %}
  <div class="projects-grid">
    {% for project in sorted_projects %}
      {% include projects.liquid %}
    {% endfor %}
  </div>
{% endif %}
</div>
