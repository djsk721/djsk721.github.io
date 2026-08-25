---
layout: page
title: Categories
description: Browse posts grouped by category.
permalink: /categories/
---

{% assign sorted_categories = site.categories | sort %}

<div class="categories-archive">
  <p class="categories-filter-status" aria-live="polite">전체 카테고리와 글을 보고 있습니다.</p>

  <div class="category-filter-bar" role="group" aria-label="카테고리 필터">
    <button type="button" class="chip filter-chip is-active" data-cat-filter="" aria-pressed="true">
      전체 <span class="filter-count">{{ site.posts | size }}</span>
    </button>
    {% for category in sorted_categories %}
      {% assign category_name = category[0] %}
      {% assign category_id = category_name | slugify %}
      {% assign category_posts = category[1] %}
      <button type="button" class="chip filter-chip" data-cat-filter="{{ category_id }}" aria-pressed="false">
        {{ category_name }} <span class="filter-count">{{ category_posts | size }}</span>
      </button>
    {% endfor %}
  </div>

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
  var archive = document.querySelector('.categories-archive');

  if (!archive) {
    return;
  }

  var sections = Array.from(archive.querySelectorAll('.category-section'));
  var status = archive.querySelector('.categories-filter-status');
  var filterChips = Array.from(archive.querySelectorAll('.filter-chip'));
  var filterLinks = Array.from(document.querySelectorAll('[data-category-filter]'));
  var allLink = document.querySelector('.sidebar-category-link-all');

  function syncChips(activeId) {
    filterChips.forEach(function(chip) {
      var isActive = chip.dataset.catFilter === activeId;
      chip.classList.toggle('is-active', isActive);
      chip.setAttribute('aria-pressed', String(isActive));
    });
  }

  function updateFilter() {
    var activeFilter = decodeURIComponent(window.location.hash.replace(/^#/, ''));
    var hasMatch = Boolean(activeFilter) && sections.some(function(section) {
      return section.dataset.categoryId === activeFilter;
    });
    var activeId = hasMatch ? activeFilter : '';
    var matchedName = '';

    sections.forEach(function(section) {
      var isMatch = !activeId || section.dataset.categoryId === activeId;
      section.hidden = !isMatch;

      if (isMatch && !matchedName) {
        var heading = section.querySelector('h2');
        matchedName = heading ? heading.textContent.trim() : activeId;
      }
    });

    archive.dataset.activeFilter = activeId;
    syncChips(activeId);

    if (status) {
      if (activeId) {
        status.textContent = '「' + matchedName + '」 카테고리의 글을 보고 있습니다.';
      } else {
        status.textContent = '전체 카테고리와 글을 보고 있습니다.';
      }
    }

    filterLinks.forEach(function(link) {
      link.classList.toggle('is-active', activeId && link.dataset.categoryFilter === activeId);
    });

    if (allLink) {
      allLink.classList.toggle('is-active', !activeId);
    }
  }

  filterChips.forEach(function(chip) {
    chip.addEventListener('click', function() {
      var target = chip.dataset.catFilter;
      if (target) {
        window.location.hash = target;
      } else if (window.location.hash) {
        history.pushState(null, '', window.location.pathname);
        updateFilter();
      }
    });
  });

  window.addEventListener('hashchange', updateFilter);
  window.addEventListener('popstate', updateFilter);
  updateFilter();
})();
</script>
