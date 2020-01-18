---
permalink: /posts/
title: "Posts"
author_profile: true
header:
  image: "/images/derpyhooves.png"
---

### 2020

[OSCP Review] (/posts/2020-01-18-OSCP-Review)

{% for post in site.posts %} {% assign currentdate = post.date | date: "%Y" %} {% if currentdate != date %}
{{ currentdate }}
{% assign date = currentdate %} {% endif %} {{ post.title }} {{ post.excerpt }} {% endfor %}