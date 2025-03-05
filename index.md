---
title: オンラインでホストされる手順
permalink: index.html
layout: home
---

# コンテンツ ディレクトリ

各ラボの演習とデモへのハイパーリンクを以下に示します。

## ラボ \(Entra ID\)

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs_EntraID'" %}
| モジュール | ラボ |
| --- | --- | 
{% for activity in labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

## ラボ \(AD DS\)

必要なラボ ファイルは、[こちらからダウンロード](https://github.com/MicrosoftLearning/AZ-140-Configuring-and-Operating-Microsoft-Azure-Virtual-Desktop/archive/master.zip)できます

{% assign labs = site.pages | where_exp:"page", "page.url contains '_ADDS'" %}
| モジュール | ラボ |
| --- | --- | 
{% for activity in labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

## ラボ \(Entra DS\)

必要なラボ ファイルは、[こちらからダウンロード](https://github.com/MicrosoftLearning/AZ-140-Configuring-and-Operating-Microsoft-Azure-Virtual-Desktop/archive/master.zip)できます

{% assign labs = site.pages | where_exp:"page", "page.url contains '_AADDS'" %}
| モジュール | ラボ |
| --- | --- | 
{% for activity in labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

<!--
## Demos

{% assign demos = site.pages | where_exp:"page", "page.url contains '/Instructions/Demos'" %}
| Module | Demo |
| --- | --- | 
{% for activity in demos  %}| {{ activity.demo.module }} | [{{ activity.demo.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
-->
