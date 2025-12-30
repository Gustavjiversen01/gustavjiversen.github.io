---
layout: page
title: Blog
subtitle: Thoughts, tutorials, and explorations
permalink: /blog/
---

<ul class="post-list">
{% for post in site.posts %}
  <li class="post-item">
    <h3 class="post-title"><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <p class="post-meta">{{ post.date | date: "%B %-d, %Y" }}</p>
    {% if post.excerpt %}
    <p class="post-excerpt">{{ post.excerpt | strip_html | truncatewords: 40 }}</p>
    {% endif %}
  </li>
{% endfor %}
</ul>

{% if site.posts.size == 0 %}
*Posts coming soon.*
{% endif %}
