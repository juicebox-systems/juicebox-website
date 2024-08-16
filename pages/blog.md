---
layout: page
title: Blog
permalink: /blog/
---

{% for post in site.posts %}
  {%- include post.html show_summary=true -%}
  {%- if post != site.posts.last -%}
    <div class='post-divider'></div>
  {%- endif -%}
{% endfor %}
