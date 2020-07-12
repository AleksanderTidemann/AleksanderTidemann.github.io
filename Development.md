---
layout: page
title: Development
permalink: /Development/
---

{%- if site.posts.size > 0 -%}
  <!-- <h2 class="post-list-heading">{{ page.list_title | default: "Posts" }}</h2> -->
  <ul class="post-list">
    {%- for post in site.categories.Development -%}
    <li>
      {%- if post.image -%}
      <img src="{{ post.image | prepend: site.baseurl }}" alt="{{ post.title }}" title="{{ post.title }}">
      {%- endif -%}
      {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
      <h3 align="left">
        <a class="post-link" href="{{ post.url | relative_url }}">
          {{ post.title | escape }}
        </a>
      </h3>
      <span class="post-meta">{{ post.date | date: date_format }}</span>
      <span class="post-meta">• {{ post.author }}</span>
      <br />
      {%- if site.show_excerpts -%}
        {{ post.excerpt }}
      {%- endif -%}
    </li>
    {%- endfor -%}
  </ul>
{%- endif -%}

<!-- Hack from https://github.com/jekyll/jekyll/issues/2538 -->
{% capture difference %} {{ site.posts | size | minus:1 }} {% endcapture %}
<!-- {% unless difference contains '-' %} -->
<!-- ***No posts... yet.*** -->
 <!-- Your code will now be dependent on page.tags being empty -->
<!-- {% endunless %} -->
