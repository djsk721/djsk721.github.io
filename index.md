---
layout: default
permalink: /
---

<section class="hero">
  <h2>Hello World</h2>
  <p>잡다한 기술 블로그.</p>
</section>

<div class="posts-grid">
  {% for post in site.posts %}
    <article class="post-card">
      <header class="post-card-header">
        <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
        <time datetime="{{ post.date | date_to_xmlschema }}">
          {{ post.date | date: "%Y. %m. %d." }}
        </time>
      </header>

      <p class="post-excerpt">{{ post.description }}</p>

      {% if post.tags %}
      <div class="post-tags">
        {% for tag in post.tags %}
          <span class="tag">#{{ tag }}</span>
        {% endfor %}
      </div>
      {% endif %}
    </article>
  {% endfor %}
</div>
