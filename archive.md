---
layout: page
title: Archive
last_modified_at: 2020-05-08 10:30:00 +0000
permalink: /archive/
---

<ul>
  {% for post in site.posts %}
    <li>
        <span class="date">{{ post.date | date: '%Y-%m-%d' }}</span>
        <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
