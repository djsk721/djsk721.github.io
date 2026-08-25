---
layout: page
title: Categories
description: Browse posts grouped by category.
permalink: /categories/
---

{% assign sorted_categories = site.categories | sort %}

<div class="categories-archive">
  <p class="categories-summary">총 {{ site.posts | size }}개의 글 · {{ sorted_categories.size }}개의 카테고리</p>
  <p class="categories-filter-status" aria-live="polite">전체 카테고리와 글을 보고 있습니다.</p>
  {% if sorted_categories.size > 0 %}
    {% for category in sorted_categories %}
      {% assign category_name = category[0] %}
      {% assign category_id = category_name | slugify %}
      {% assign category_posts = category[1] %}
      <section class="category-section" id="{{ category_id }}" data-category-id="{{ category_id }}">
        <header class="category-section-header">
          <h2>{{ category_name }}</h2>
          <p>{{ category_posts | size }}개의 글</p>
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
    <p class="categories-empty">아직 카테고리가 없습니다.</p>
  {% endif %}
</div>

<script>
(function() {
  const archive = document.querySelector('.categories-archive');

  if (!archive) {
    return;
  }

  const sections = Array.from(archive.querySelectorAll('.category-section'));
  const status = archive.querySelector('.categories-filter-status');
  const filterLinks = Array.from(document.querySelectorAll('[data-category-filter]'));
  const allLink = document.querySelector('.sidebar-category-link-all');

  function updateFilter() {
    const activeFilter = decodeURIComponent(window.location.hash.replace(/^#/, ''));
    const hasMatch = Boolean(activeFilter) && sections.some(function(section) {
      return section.dataset.categoryId === activeFilter;
    });
    let matchedName = '';

    sections.forEach(function(section) {
      const isMatch = !hasMatch || section.dataset.categoryId === activeFilter;
      section.hidden = !isMatch;

      if (isMatch && !matchedName) {
        const heading = section.querySelector('h2');
        matchedName = heading ? heading.textContent.trim() : activeFilter;
      }
    });

    archive.dataset.activeFilter = hasMatch ? activeFilter : '';

    if (status) {
      if (hasMatch) {
        status.textContent = '「' + matchedName + '」 카테고리의 글을 보고 있습니다.';
      } else {
        status.textContent = '전체 카테고리와 글을 보고 있습니다.';
      }
    }

    filterLinks.forEach(function(link) {
      link.classList.toggle('is-active', hasMatch && link.dataset.categoryFilter === activeFilter);
    });

    if (allLink) {
      allLink.classList.toggle('is-active', !hasMatch);
    }
  }

  window.addEventListener('hashchange', updateFilter);
  updateFilter();
})();
</script>
