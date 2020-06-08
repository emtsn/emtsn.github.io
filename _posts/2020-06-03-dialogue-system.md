---
layout: post
title:  "Dialogue System"
description: Quick demo of my Dialogue System in Unity.
date: 2020-06-03
unity_dir: SODialogue-WebGL
categories: projects
tags: Unity C#
---

# Quick Demo of My Dialogue System (Unity 2019.3)

Dialogue System with characers, items, variables, choices, and paths stored using ScriptableObjects with a custom editor.

{% comment %}
Originally created in 2019.2, but 2019.3 introduced the new SerializeReference attribute which made for easier serialization. Since this also meant events could be edited in the inspector by default, I scrapped my own list layout in favour of Unity's ReorderableList.
{% endcomment %}

Here is what a single dialogue block looks like in the editor:  
![Example Image for the Editor](/assets/ExampleImage.png)

Detailed Features:
- Dialogue Box
    - Has manual, auto, and skip options
        - Manual: must click for the next line
        - Auto: will automatically click for the next line
        - Skip: skip through the dialogue quickly
    - The text log will log the lines that appear in the box
- Characters
    - Characters with a set portrait and voice tone (the blips)
    - Portraits will appear when they speak and dim when they are not speaking
    - Can change portrait positions during dialogue
- Items
    - Items can be given and taken from players
- Variables
    - Variables can be global, local, or temporary
        - Global variables will stay forever
        - Local variables get reset when the dialogue stops
        - Temporary variables get reset when a dialogue block finishes
    - Variables can be operated on by or set from values or other variables
    - Can generate random values
- Choices
    - Choices have a set dialogue block to go to and a label for the UI
    - Choices can be hidden or dimmed if checks on variables do not pass
- Paths
    - Paths have a set dialogue block to go to
    - A path with the variables checks passing will automatically be picked to go to
- Jumps
    - A way of creating control flow inside of a dialogue block
    - The 'finish' line will skip control to the end

Art made in [Aseprite](https://github.com/aseprite/aseprite/).  
Sounds made in [Bfxr](https://github.com/increpare/bfxr).
