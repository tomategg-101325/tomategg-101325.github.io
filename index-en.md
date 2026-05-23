---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page-en
title: "Homepage"
permalink: "/en/"
categories: en-us-page
---

Welcome to my personal website! The site contains tech blogs and my own thought pieces 😺

You are also welcome to visit the [About](/en/about) page for more information about me.

🇨🇳 前往中文主页，请访问[此处](/index/)。

## Posts

<ul>
  {% for post in site.categories.en-us %}
    <li>
      <h4><a href="{{ post.url }}">{{ post.title }}</a></h4>
      <p style="color:gray">Tags: {{ post.tags | sort }}</p>
      <p>{{ post.excerpt }}</p>
      <hr>
    </li>
  {% endfor %}
</ul>
