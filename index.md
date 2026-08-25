---
layout: default
permalink: /
---

<section class="hero">
  <h2>Hello World</h2>
  <p>잡다한 기술 블로그.</p>
  <p class="hero-meta">총 {{ site.posts | size }}개의 글 · {{ site.categories | size }}개의 카테고리</p>
</section>

<h2 class="section-label">최근 글</h2>

<ul class="post-list">
  {% for post in site.posts %}
  <li class="post-row">
    <a class="post-row-link" href="{{ post.url | relative_url }}">
      <div class="post-row-header">
        {% if post.categories %}
        <span class="chip chip--category">{{ post.categories.first }}</span>
        {% endif %}
        <h3 class="post-row-title">{{ post.title }}</h3>
        <time class="post-row-date" datetime="{{ post.date | date_to_xmlschema }}">
          {{ post.date | date: "%Y. %m. %d." }}
        </time>
      </div>
      {% if post.description %}
      <p class="post-row-excerpt">{{ post.description }}</p>
      {% endif %}
      {% if post.tags %}
      <div class="post-row-tags">
        {% for tag in post.tags limit: 2 %}
        <span class="tag">#{{ tag }}</span>
        {% endfor %}
        {% if post.tags.size > 2 %}
        <span class="tag tag-more">+{{ post.tags.size | minus: 2 }}</span>
        {% endif %}
      </div>
      {% endif %}
    </a>
  </li>
  {% endfor %}
</ul>
