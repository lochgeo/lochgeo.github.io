---
layout: page
permalink: /archive/
title: Archive
---

<div class="archive">
  {% assign years = site.posts | group_by_exp: 'post', 'post.date | date: "%Y"' %}
  {% for year in years reversed %}
  <section class="archive-group">
    <h2 class="archive-year">{{ year.name }}<span class="archive-count">{{ year.items.size }} post{% if year.items.size != 1 %}s{% endif %}</span></h2>
    <ul class="archive-list">
      {% for post in year.items %}
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
</div>
