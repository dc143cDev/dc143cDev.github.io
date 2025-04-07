---
layout: default
title: Projects
description: dc143c's projects
---

<div class="projects-container">
  <h1 class="projects-title">Projects</h1>
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
          <span class="project-date">{{ project.date | date: "%b %Y" }}</span>
        </div>
        
        {% if project.description %}
          <p class="project-description">{{ project.description }}</p>
        {% endif %}
      </a>
    {% endfor %}
  </div>
</div>

<script>
document.addEventListener('DOMContentLoaded', function() {
  // 페이지 로드 완료 후 실행되는 스크립트
  // 프로젝트 카드에 애니메이션 클래스가 이미 CSS에 적용되어 있으므로
  // 추가 스크립트는 필요하지 않습니다.
});
</script>