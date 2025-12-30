---
layout: default
title: Home
---

# Hi, I'm Gustav

I'm an ML student building things and writing about what I learn.

I'm interested in machine learning, mathematics, and building useful tools. Here you'll find my [projects](/projects), [writing](/blog), and ways to connect.

## Recent Writing

<ul class="post-list">
{% for post in site.posts limit:3 %}
  <li class="post-item">
    <h3 class="post-title"><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <p class="post-meta">{{ post.date | date: "%B %-d, %Y" }}</p>
    {% if post.excerpt %}
    <p class="post-excerpt">{{ post.excerpt | strip_html | truncatewords: 30 }}</p>
    {% endif %}
  </li>
{% endfor %}
</ul>

{% if site.posts.size > 3 %}
[See all posts →](/blog)
{% endif %}

## Featured Projects

<ul class="project-grid">
{% for project in site.projects limit:3 %}
  <li class="project-card">
    <h3 class="project-title"><a href="{{ project.url }}">{{ project.title }}</a></h3>
    <p class="project-description">{{ project.description }}</p>
    {% if project.tags %}
    <div class="project-tags">
      {% for tag in project.tags limit:3 %}
      <span class="tag">{{ tag }}</span>
      {% endfor %}
    </div>
    {% endif %}
  </li>
{% endfor %}
</ul>

{% if site.projects.size > 3 %}
[See all projects →](/projects)
{% endif %}
