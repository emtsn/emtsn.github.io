---
layout: gamepost
title:  "Dialogue System"
date: 2020-06-03
unity_dir: SODialogue-WebGL
categories: projects
tags: Unity C#
---

# Demo of My Dialogue System (Unity 2019.3)

Dialogue System with characers, items, choices, variables, and paths stored using ScriptableObjects with a custom editor.

{% comment %}
Originally created in 2019.2, but 2019.3 introduced the new SerializeReference attribute which made for easier serialization. Since this also meant events could be edited in the inspector by default, I scrapped my own list layout in favour of Unity's ReorderableList.
{% endcomment %}

Here is what it looks like in the editor:  
![Example Image for the Editor](/assets/ExampleImage.png)

Art made in Aseprite.  
Sounds made in Bfxr.
