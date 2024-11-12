---
layout: default
title: Projects
permalink: /projects/
---

<div class="posts-container">
  <h1 class="archive-title">Projects</h1>
  <div class="posts-list">
    {% for post in site.projects %}
      <article class="post-item" data-tags="{{ post.tags | join: ',' | downcase }}">
        <h3 class="post-title">
          <a href="{{ post.url }}">{{ post.title }}</a>
        </h3>
        <div class="post-meta">
          <time>{{ post.date | date: "%Y-%m-%d" }}</time>
          {% if post.tags %}
            <div class="post-tags">
              {% for tag in post.tags %}
                <span class="tag-small">{{ tag }}</span>
              {% endfor %}
            </div>
          {% endif %}
        </div>
      </article>
    {% endfor %}
  </div>
</div>