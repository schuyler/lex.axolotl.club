---
layout: default
title: Home
---

# Welcome

This is a space for polished explorations of AI architecture, cognitive systems, and the intersection of theory and practice. 

The wiki holds the raw notes, ongoing research, and live brainstorming. Here, you'll find structured essays and breakdowns meant for sharing with friends and collaborators.

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
