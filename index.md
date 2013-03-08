---
layout: page
title: Welcome to Zeutrap's Collections!
---
{% include JB/setup %}

<h4> Here's my posts list. </h4>

<table class="table table-hover ">
  {% for post in site.posts %}
    <tr><th><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</th><th>{{ post.date | date_to_string }}</th></tr>
  {% endfor %}
</table>
