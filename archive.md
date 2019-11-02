---
layout: page
title: Archive
last_modified_at: 2019-11-01 18:57:00 +0000
permalink: /archive/
---


<ul>
  {% for post in site.posts %}
    <li>
        <span class="date">{{ post.date | date: '%d/%m/%Y' }}</span>
        <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
