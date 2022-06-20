---
title: Checkoff
permalink: /unity/unity_for_teens_3
key: unity-unity_for_teens_3
layout: article
nav_key: unity
sidebar:
  nav: unity
license: false
aside:
  toc: true
show_edit_on_github: false
show_date: false
---

Implement a *Powerup Cast* feature using the ScriptableObject Event system. 
* When key Z is pressed, attempt to cast powerup in the first slot. Nothing should happen if you haven't collected anything there. Else, it must affect Mario's max or jump speed for a *specific duration* as dictated in the Powerup ScriptableObject instance. 
* Similarly with key X, for the powerup in the second slot. 
* You need to have at least **two different powerups** as per previous lab. 

To do this, will need to create another `CastEventListener.cs` and its matching `CastEvent` scriptable object. It works with custom `UnityEvent<KeyCode>` since we need want to pass the `Keycode` to the listener attached at `PowerupManager` and know exactly whether the first slot or the second slot is casted. The diagram below shows a possible architecture:

<img src="https://www.dropbox.com/s/0qrp8a8uwm88xdn/31.png?raw=1"  class="center_ninety"/>

The gif below summarises the end result:
* You can carry over unused powerup to the next scene
* Score is carried over
* Usage of powerup should dissipate after a  `duration`
* There's more than 1 kind of powerup
* Restart functionality is **optional**, but cool to have and not easy to implement. Simply implement another GameEventListener for OnPlayerDeath that shows the panel and restart button, and callback to reload **this scene** again.  
* You may use Singleton pattern and totally ignore the ScriptableObject architecture if you want

<img src="https://www.dropbox.com/s/b9p1tw128c117ma/checkoff.gif?raw=1"  class="center_ninety"/>
