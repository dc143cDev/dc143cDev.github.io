---
layout: default
---

<div class="projects-container">
  <div class="projects-list">
    {% for project in site.projects %}
      <a href="{{ project.url }}" class="project-card">
        {% if project.thumbnail %}
          <div class="project-thumbnail">
            <img src="{{ project.thumbnail }}" alt="{{ project.title }}">
          </div>
        {% else %}
          <div class="project-thumbnail no-image">
            <div class="no-image-placeholder">
              <span>{{ project.title | slice: 0, 1 }}</span>
            </div>
          </div>
        {% endif %}
        
        <h3 class="project-title">{{ project.title }}</h3>
        <div class="project-meta">
          <span class="project-type">{{ project.type }}</span>
          <span class="project-date">{{ project.date | date: "%Y년 %m월" }}</span>
        </div>
        
        {% if project.description %}
          <p class="project-description">{{ project.description }}</p>
        {% endif %}
      </a>
    {% endfor %}
  </div>
</div>