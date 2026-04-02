---
layout: page
title: Categories
description: Browse posts grouped by category.
permalink: /categories/
---

{% assign sorted_categories = site.categories | sort %}

<div class="categories-archive">
  {% if sorted_categories.size > 0 %}
    {% for category in sorted_categories %}
      {% assign category_name = category[0] %}
      {% assign category_posts = category[1] %}
      <section class="category-section" id="{{ category_name | slugify }}">
        <header class="category-section-header">
          <h2>{{ category_name }}</h2>
          <p>{{ category_posts | size }} posts</p>
        </header>

        <ul class="category-post-list">
          {% for post in category_posts %}
            <li class="category-post-item">
              <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
              <time datetime="{{ post.date | date_to_xmlschema }}">
                {{ post.date | date: "%Y. %m. %d." }}
              </time>
            </li>
          {% endfor %}
        </ul>
      </section>
    {% endfor %}
  {% else %}
    <p class="categories-empty">No categories have been added yet.</p>
  {% endif %}
</div>
