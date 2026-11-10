---
layout: default
title: Home
---

# Posts

<ul class="post-list">
{% for post in site.posts %}
  <li>
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    <p>{{ post.excerpt | markdownify | strip_html | truncatewords: 30 }}</p>
    <p><a href="{{ post.url | relative_url }}">Read more →</a></p>
  </li>
{% endfor %}
</ul>
