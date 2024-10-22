---
layout: page
title: Projects
---

{% for project in site.pages %}
  {% if project.path contains 'projects/' %}
    <h2><a href="{{ project.url | relative_url }}">{{ project.title }}</a></h2>
    <p>{{ project.excerpt }}</p>
  {% endif %}
{% endfor %}