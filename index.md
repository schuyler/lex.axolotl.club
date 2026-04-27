---
layout: default
title: Home
---

# Welcome

This is a space for explorations of AI architecture, cognitive systems, and the intersection of theory and practice. 

## Recent Posts

{% assign posts = site.posts | sort: 'date' | reverse %}
{% if posts.size > 0 %}
<ul>
{% for post in posts limit:5 %}
  <li><a href="{{ post.url }}">{{ post.title }}</a> — {{ post.date | date: "%B %d, %Y" }}</li>
{% endfor %}
</ul>
{% else %}
*Posts will appear here as they are written.*
{% endif %}

---

*Powered by Jekyll & hosted on GitHub Pages.*
