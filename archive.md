---
layout: default
title: Archive
permalink: /archive/
---

<div class="posts-container">
  <h1 class="archive-title">Archive</h1>
  <p class="results-count" style="color: var(--gray-600); margin-top: -1rem; font-size: 1rem;">
    {{ site.posts.size }} items
  </p>

  <div class="tags-filter">
    {% assign all_tags = site.posts | map: "tags" | compact | flatten | uniq | sort %}
    {% for tag in all_tags %}
      <span class="tag" data-tag="{{ tag | downcase }}">
        {{ tag }}
      </span>
    {% endfor %}
  </div>

  <h1 class="results-title" style="display: none;"><span id="resultCount"></span> Results</h1>

  <div class="posts-list">
    {% for post in site.posts %}
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

<script>
document.addEventListener('DOMContentLoaded', function() {
  const tags = document.querySelectorAll('.tags-filter .tag');
  const posts = document.querySelectorAll('.post-item');
  const archiveTitle = document.querySelector('.archive-title');
  const resultsCount = document.querySelector('.results-count');
  const totalPosts = posts.length;
  
  // 초기 아이템 개수 표시
  resultsCount.textContent = `${totalPosts} items`;
  
  tags.forEach(tag => {
    tag.addEventListener('click', function() {
      const selectedTag = this.dataset.tag;
      const tagText = this.textContent.trim();
      
      if (this.classList.contains('active')) {
        // 태그 해제
        this.classList.remove('active');
        posts.forEach(post => post.style.display = 'block');
        
        // 타이틀 변경
        archiveTitle.textContent = 'Archive';
        resultsCount.textContent = `${totalPosts} items`;
      } else {
        // 태그 선택
        tags.forEach(t => t.classList.remove('active'));
        this.classList.add('active');
        
        let visibleCount = 0;
        posts.forEach(post => {
          const postTags = post.dataset.tags.split(',');
          if (postTags.includes(selectedTag)) {
            post.style.display = 'block';
            visibleCount++;
          } else {
            post.style.display = 'none';
          }
        });
        
        // 타이틀 변경
        archiveTitle.textContent = tagText;
        resultsCount.textContent = `${visibleCount} items`;
      }
    });
  });
});
</script>
