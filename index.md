---
title: Azure AI 视觉练习
permalink: index.html
layout: home
---

# Azure AI 视觉练习

以下练习旨在支持 Microsoft Learn 上的模块。


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %}

{% for activity in labs  %} {% if activity.url contains 'ai-foundry' %} {% continue %} {% endif %}
  - [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
