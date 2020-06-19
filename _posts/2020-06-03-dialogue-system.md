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

Originally created Summer 2019 for Unity 2019.2, but updated to utilize new features that were added with Unity 2019.3.

Here is what a single dialogue block looks like in the editor:  
![Example Image for the Editor](/assets/ExampleImage.png)

Detailed Features:
- Dialogue Box
    - Has manual, auto, and skip options
        - Manual: must click for the next line
        - Auto: will automatically click for the next line
        - Skip: skip through the dialogue quickly
    - Supports TextMesh Pro (i.e. in-text tags for bold, italics, etc...)
    - Logs any lines of dialogue that appear
- Characters
    - Have a set portrait and voice tone (the blips)
    - Portraits will appear and focus on speaking characters 
    - Can change portrait positions during dialogue
- Items
    - Can be given or taken from players
- Variables
    - Can be global, local, or temporary
        - Global variables will stay forever
        - Local variables get deleted when the dialogue stops
        - Temporary variables get deleted when a dialogue block finishes
    - Can be operated on by or set from values or other variables
    - Can generate random values
- Choices
    - Have a set dialogue block to go to and a custom label
    - Can be hidden or dimmed if checks on variables do not pass
- Paths
    - Have a set dialogue block to go to
    - A path with the variables checks passing will automatically be picked to go to
- Jumps
    - Creates control flow inside of a dialogue block
    - The 'finish' line will skip control to the end

Art made in [Aseprite](https://github.com/aseprite/aseprite/).  
Sounds made in [Bfxr](https://github.com/increpare/bfxr).
