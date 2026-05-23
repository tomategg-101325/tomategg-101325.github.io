---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page-cn
title: "主页"
categories: zh-cn-page
---

欢迎来到我的个人网站！本站会盛放一些技术博客和自己的碎碎念😺

大家也可以访问[关于](/about/)页面，更深入地了解我。

🇬🇧 Click [here](/en/) for the English site.

## 帖子

<ul>
  {% for post in site.categories.zh-cn %}
    <li>
      <h4><a href="{{ post.url }}">{{ post.title }}</a></h4>
      <p style="color:gray">标签：{{ post.tags | sort }}</p>
      <p>{{ post.excerpt }}</p>
      <hr>
    </li>
  {% endfor %}
</ul>

