---
layout: page
title: Categories
description: Browse posts grouped by category.
permalink: /categories/
---

{% assign sorted_categories = site.categories | sort %}

<div class="categories-archive">
  <p class="categories-filter-status" aria-live="polite">Showing all categories.</p>
  {% if sorted_categories.size > 0 %}
    {% for category in sorted_categories %}
      {% assign category_name = category[0] %}
      {% assign category_id = category_name | slugify %}
      {% assign category_posts = category[1] %}
      <section class="category-section" id="{{ category_id }}" data-category-id="{{ category_id }}">
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
    let matchedCount = 0;

    sections.forEach(function(section) {
      const isMatch = !activeFilter || section.dataset.categoryId === activeFilter;
      section.hidden = !isMatch;

      if (isMatch) {
        matchedCount += 1;
      }
    });

    archive.classList.toggle('is-filtering', Boolean(activeFilter) && matchedCount > 0);
    archive.dataset.activeFilter = matchedCount > 0 ? activeFilter : '';

    if (status) {
      if (!activeFilter || matchedCount === 0) {
        status.textContent = 'Showing all categories.';
      } else {
        status.textContent = 'Showing category: ' + activeFilter + '.';
      }
    }

    filterLinks.forEach(function(link) {
      link.classList.toggle('is-active', link.dataset.categoryFilter === activeFilter && matchedCount > 0);
    });

    if (allLink) {
      allLink.classList.toggle('is-active', !activeFilter || matchedCount === 0);
    }
  }

  window.addEventListener('hashchange', updateFilter);
  updateFilter();
})();
</script>
