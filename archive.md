---
layout: page
title: Archive
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
