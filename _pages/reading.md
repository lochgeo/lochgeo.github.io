---
layout: page
permalink: /reading/
title: Reading List
---

<div class="archive">
  {% assign years = site.reading | sort: 'date' | reverse | group_by_exp: 'item', 'item.date | date: "%Y"' %}
  {% for year in years %}
  <section class="archive-group">
    <h2 class="archive-year">{{ year.name }}<span class="archive-count">{{ year.items.size }} entr{% if year.items.size == 1 %}y{% else %}ies{% endif %}</span></h2>
    {% assign months = year.items | group_by_exp: 'item', 'item.date | date: "%B"' %}
    {% for month in months %}
    <h3 class="archive-month">{{ month.name }}</h3>
    <ul class="archive-list">
      {% for item in month.items %}
      <li class="archive-item">
        <a href="{{ site.baseurl }}{{ item.url }}">
          <time datetime="{{ item.date | date_to_xmlschema }}">{{ item.date | date: '%b %-d' }}</time>
          <span class="archive-title">{% if item.title and item.title != "" %}{{ item.title }}{% else %}{{ item.content | strip_html | truncate: 80 }}{% endif %}</span>
        </a>
      </li>
      {% endfor %}
    </ul>
    {% endfor %}
  </section>
  {% endfor %}
</div>