---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page-cn
title: "主页"
categories: zh-cn-page
---

# 欢迎！

嗨，欢迎来到我的个人网站！本站会盛放一些技术博客和自己的碎碎念😺<br>大家也可以访问[关于](/about/)页面，更深入地了解我。

[订阅](/feed.xml) RSS。

<br><br>

# 帖子

<ul>
  {% for post in site.categories.zh-cn %}
    <li>
      <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
      <p style="color:gray">
        <img src="{{ '/assets/icons/calendar.png' | relative_url }}" title="日期" alt="日期" loading="lazy" width="10" height="10" style="aspect-ratio: 100 / 100;"> {{ post.date | date: "%Y-%m-%d" }}&nbsp;&nbsp;&nbsp;&nbsp;<img src="{{ '/assets/icons/tag.png' | relative_url }}" title="标签" alt="标签" loading="lazy" width="10" height="10" style="aspect-ratio: 100 / 100;"> {{ post.tags | join: "、" }}
      </p>
      <p>{{ post.excerpt }}</p>
      <hr>
    </li>
  {% endfor %}
</ul>

