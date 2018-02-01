---
layout: default
avatar: true
permalink: /
---
# 최근 포스트

{% for post in site.posts limit: 2 %}
  <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
{% endfor %}

---
