---
layout: page
permalink: /categories/
title: Categories
---

{% assign sorted_categories = site.categories | sort %}
<nav class="category-cloud" aria-label="Categories">
  {% for category in sorted_categories %}
    <a class="category-pill" href="#{{ category[0] | slugize }}">{{ category[0] }} <span class="category-pill-count">{{ category[1].size }}</span></a>
  {% endfor %}
</nav>

{% for category in sorted_categories %}
<section class="category-group">
  <h2 class="category-head">
    {{ category[0] }}
    <span class="archive-count">{{ category[1].size }} post{% if category[1].size != 1 %}s{% endif %}</span>
  </h2>
  <ul class="archive-list">
    {% assign posts = category[1] | sort: "date" | reverse %}
    {% for post in posts %}
    <li class="archive-item">
      <a href="{{ site.baseurl }}{{ post.url }}">
        <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: '%b %-d' }}</time>
        <span class="archive-title">{% if post.title and post.title != "" %}{{ post.title }}{% else %}{{ post.excerpt | strip_html | truncate: 80 }}{% endif %}</span>
      </a>
    </li>
    {% endfor %}
  </ul>
</section>
{% endfor %}
