---
title: Checkoff
permalink: /unity/unity_for_children_4
key: unity-unity_for_children_4
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



The gif summarises what you need for checkoff. There's a lot of things that need to be done so read each point carefully. 
1. **Use the ObjectPooler** and *spawn* 2 enemies at a time. The Player can "Kill" the enemy any way you want, such as stepping from the top. This will **increase score** and spawn **one new enemy**.  Make sure you show that you use an object pooler by recording the *Hierarchy* as well. 
2. Create some "**interactive**" **bricks** to collect coins or whatever rewards that will **increase score** but *spawn one new enemy*. It is up to you to determine how many max enemies can be present at a time. In the gif below, we limit total number of goomba enemies and green enemies to be 5. 
3. Player will "**die**" under certain conditions (up to you), and this will cause the enemy to "**rejoice**".  Both actions of player dying and enemy rejoicing must be clear to the player. In the gif sample below, player will die if it collides with the enemy from anywhere except from the top. The enemies have this cute little dance when the player dies, and player's death is animated as well. 
4. Use **AudioMixer** in your project, in any way you want. 
5. **Powerups**: create two different powerups from anywhere e.g: the question boxes. Player can "collect" it as shown, and "cast" it later. Bind the "casting" to two distinct keys, signifying that the player *consumes* it. The effect should disappear after a fixed number of seconds. For example, the red mushroom allows the player to jump higher for 5 seconds. The max number of powerups that can be collected can be fixed to 1 for each type for simplicity. 
6. Usage of **ScriptableObject** to store some unchanged constants like enemy speed, fixed patrol locations, etc. This won't be visible easily in the game, so you need to record yourself clicking to the scriptable object instance and **show** that the values are used by other scripts. 

<img src="https://www.dropbox.com/s/b2nork7nh0g9f6d/checkoff.gif?raw=1"  class="center_ninety"/>

It is alright if your screen recording for this checkoff takes more time. 


What to do next?
{:.error}

In the next lesson, we will further polish our game by learning how to transition between Scenes and using an alternative event-based Architecture using **Scriptable Objects**. This architecture is useful to **eliminate** dependency between Scenes, and we can use it to implement basic stuffs in game such as Inventory System and Combo System. We will also learn another way to perform asynchronous execution using async and await, which is fundamentally different from Coroutines, along with some issues to watch out for when using them such as *memory leak*. 







 

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5MzYxNzE0MjQsLTE4MDk0ODI1NjksNj
gyNTI3MzYwLC0xNTMzNDA1OTgsMTE5NjQ2MDk2MiwtNjk5OTMy
MDUxLC0zMDUxMDgwODgsNjAzMTcxOTQwXX0=
-->