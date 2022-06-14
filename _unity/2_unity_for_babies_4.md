---
title: Checkoff
permalink: /unity/unity_for_babies_4
key: unity-unity_for_babies_4
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


For this checkoff, you're required to implement everything you can see in the demo .gif below (you can just gauge and estimate the placement of each obstacle). 


Everything is mostly covered in this lab, **except the script that controls the `ConsumableMushroomSimple`** and the  addition of **background objects** as shown in the demo .gif below. In summary, for this checkoff you must (but you can do more):
* Write a script so the spawned mushroom can behave exactly like the .gif shown.

	* You can use the suggested method above, or use other means that gives the same visual effect. You can also use any Mushroom sprite, it doesn't have to be this green one or the red one above. 
	* Note that you are also **free to use any alternative method** to create the bouncy question box (by using Animator, or manually from script). You don't have to follow our method of using the SpringJoint2D. 
* Create various GameObjects to render some nice background: the small hills in the background as well as the background _wallpaper_. You have to organise your Sprite Renderer's **Sorting Layer** properly here. Obviously all these objects do not affect the game, so you'll only need to have Transform and Sprite Renderer components attached to these background GameObjects. 

> Get used to the creation and organisation of various Layers, Tags, and Sorting Layers, and understand the application of each property:
> * **Layer**: for 2D collision matrix and *Camera culling mask (next week)*
> * **Sorting Layer**: to determine what is rendered in front of what when they're on the same XY-plane
> * **Tags**: to quickly prototype and identify the `gameObject` (although you can also search by its *[name](https://docs.unity3d.com/ScriptReference/GameObject.Find.html)*). 

Refer to our course handout as usual to find out the standard protocol on how to submit your lab checkoff. 

<img src="https://www.dropbox.com/s/uhdirkzz1q9dr55/checkoff2.gif?raw=1"  class="center_ninety"/>



What to do next?
{:.error}

We improve on a few things this time round, but we still lack a few features: the enemies, counting of scores and coin collection, having power-ups effect on the character, and arranging the world to match Super Mario Bros World 1-1. However with your skills now, it should be clear how to implement them (at least to get it to work) so we will not put it as a priority at this point. In the next Lab we will learn new things instead, that is how to polish the looks of this game: adding VFX and post-processing. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2NjA4OTAxODYsMTQ3ODY4OTg5OCwtOT
YzNzk4OTk0LC0yMTEzNTY2NzY2LC04OTkyNTQ1MTAsMTM0MTk4
NjEzOSwxNDIxMTUwMzAxLDE1MzYxODMyOTAsMzI0NTk5OTAwLC
0xNzQ3ODg5MTA0LDU2MjczOTk2MiwxMTk3MjUyNTgyLC0xMzI3
MzI2MDAzLC0yMDE0ODI4NTc3LDE0MTg0NDM2NTQsLTE4OTUyOT
E1NzQsLTcwMzcxOTIyMCwtODMxNjI5MTk0LDEyODIxNDIwNjUs
LTEyODI3OTQ4MjhdfQ==
-->