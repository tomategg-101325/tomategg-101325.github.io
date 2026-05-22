---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
title: "Homepage"
permalink: "/en/"
---

🇨🇳 前往中文主页，请访问[此处](/index/)。

## Posts

<ul>
  {% for post in site.categories.enus %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
      <hr>
    </li>
  {% endfor %}
</ul>
