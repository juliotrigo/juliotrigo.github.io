---
layout: page
title: Articles
last_modified_at: 2020-05-08 18:00:00 +0000
permalink: /articles/
redirect_from: /archive/
---

<ul class="post-archive nobull">
  {% for post in site.posts %}
    <li>
        <span class="post-meta">{{ post.date | date: '%Y-%m-%d' }}</span>
        <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

{%- if site.posts.size > 0 -%}
    <ul class="post-list">
      {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
      {%- for post in site.posts -%}
      <li>
        <span class="post-meta">{{ post.date | date: date_format }}</span>
        <h2>
          <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
        </h2>

        {%- if site.show_excerpts -%}
          {%- if post.excerpt -%}
            <span>{{ post.excerpt | remove: '<p>' | remove: '</p>' }}<a href="{{ post.url | relative_url }}">{{ site.excerpt_link_text | escape }}</a></span>
          {%- endif -%}
        {%- endif -%}
      </li>
      {%- endfor -%}
    </ul>

  {%- endif -%}
