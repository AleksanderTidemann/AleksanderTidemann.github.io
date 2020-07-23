---
layout: page
title: All Topics
permalink: /Alltopics/
---

{%- if site.posts.size > 0 -%}
  <!-- <h2 class="post-list-heading">{{ page.list_title | default: "Posts" }}</h2> -->
  <ul class="post-list">
    {%- for post in site.posts -%}
    {%- if post.categories contains 'General'
    or post.categories contains 'Projects'
    -%}

    <li>
      {%- if post.image -%}
      <img src="{{ post.image | prepend: site.baseurl }}" alt="{{ post.title }}" title="{{ post.title }}">
      {%- endif -%}
      <h3 align="left">
        <a class="post-link" href="{{ post.url | relative_url }}">
          {{ post.title | escape }}
        </a>
      </h3>
      {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
      <span class="post-meta">{{ post.date | date: date_format }}</span>
      <span class="post-meta">• {{ post.author }}</span>
      <span class="post-meta"><strong>{{ post.categories | join: ", " }}</strong></span>
      <br />
      {%- if site.show_excerpts -%}
        {{ post.excerpt }}
      {%- endif -%}
    </li>
    {%- endif -%}
    {%- endfor -%}
  </ul>
  {%- else -%}
  <p>No posts yet.</p>
  {%- endif -%}
