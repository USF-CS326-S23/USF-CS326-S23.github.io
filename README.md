---
layout: home
title: Home
nav_order: 1
#nav_exclude: true
permalink: /:path/
seo:
  type: Course
  name: CS 326 Operating Systems Spring 2023
---

# CS 326 Operating Systems
Spring 2023 &nbsp; &nbsp; &nbsp; &nbsp; [Sec 01 Zoom](https://usfca.zoom.us/j/83850643096)  &nbsp; &nbsp; [Sec 02 Zoom](https://usfca.zoom.us/j/84322232934)

{% for module in site.modules %}
{{ module }}
{% endfor %}
