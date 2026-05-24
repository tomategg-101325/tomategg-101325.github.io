---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page-en
title: "Posts"
permalink: "/en/posts/"
categories: en-us-page
---

# All Posts

<ul>
  {% for post in site.categories.en-us %}
    <li>
      <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
      <p style="color:gray">
        <img src="{{ '/assets/icons/calendar.png' | relative_url }}" title="Date" alt="Date" loading="lazy" width="10" height="10" style="aspect-ratio: 100 / 100;"> {{ post.date | date: "%Y-%m-%d" }}&nbsp;&nbsp;&nbsp;&nbsp;<img src="{{ '/assets/icons/tag.png' | relative_url }}" title="Tags" alt="Tags" loading="lazy" width="10" height="10" style="aspect-ratio: 100 / 100;"> {{ post.tags | join: ", " }}
      </p>
      <p>{{ post.excerpt }}</p>
      <hr>
    </li>
  {% endfor %}
</ul>
