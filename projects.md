---
layout: page
title: Projects
subtitle: Things I've built
permalink: /projects/
---

<ul class="project-grid">
{% for project in site.projects %}
  <li class="project-card">
    <h3 class="project-title"><a href="{{ project.url }}">{{ project.title }}</a></h3>
    <p class="project-description">{{ project.description }}</p>
    {% if project.tags %}
    <div class="project-tags">
      {% for tag in project.tags %}
      <span class="tag">{{ tag }}</span>
      {% endfor %}
    </div>
    {% endif %}
  </li>
{% endfor %}
</ul>

{% if site.projects.size == 0 %}
*Projects coming soon.*
{% endif %}
