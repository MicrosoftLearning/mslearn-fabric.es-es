---
title: Instrucciones hospedadas en línea
permalink: index.html
layout: home
---

# Ejercicios de Microsoft Fabric

Los siguientes ejercicios están diseñados como apoyo para los módulos de [Microsoft Learn](https://aka.ms/learn-fabric).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Módulo | Laboratorio |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

