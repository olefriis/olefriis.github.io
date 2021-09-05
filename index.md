---
layout: default
title: "The upcoming blog of olefriis!"
---

## Nothing to see here yet

Still reading the [Jekyll docs](https://jekyllrb.com/docs/)...

<h1>Latest Posts</h1>

<ul>
  {% for post in site.posts %}
    <li>
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
