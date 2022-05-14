---
title: Unity for Toddlers
permalink: /unity/unity_for_toddlers
key: unity-unity_for_toddlers
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

**Learning Objectives:** Visual Effects

-   Adding visual effects with **URP**
- Basics of 2D lights 
-   Creating and using **shader graphs** for 2D rendering: glow and outline,
-   Adding **post processing effects**: Bloom filter
-   **Parallax background** effect: using Texture scrolling and secondary Cameras, setting culling mask
- Basics of **particle systems**: textured sheet particles
- Creating breakable prefab

# Introduction

By now, you should be more or less equipped to implement the basic game logic (at least it will work), for example create platforms, create enemies, create consumable power-ups, count scores, and restarting the game. The next important element in a game is feedback: both visual and auditory feedback. To prove this point, download the asset in the course handout and import it to your Mario project or a new project (your choice). We will be working with this side project for awhile first before returning to our Mario project. 

> The complete free asset can be downloaded from [GothicVania Church Pack](https://assetstore.unity.com/packages/2d/characters/gothicvania-church-pack-147117?aid=1101lPGj&utm_source=aff) from Unity asset score. The content of this section is obtained from the wonderful [Brackeys tutorial](https://www.youtube.com/watch?v=WiDVoj5VQ4c). 


# Universal Render Pipeline

URP package allows us to easily create optimized graphics. We need to enable it first in our project before we get started. 

## Download URP Package
Go to Window >> Package manager, and download Universal RP from Unity Registry as shown. 

<img src="https://www.dropbox.com/s/7v0dbmiccay7hu5/1.png?raw=1"  class="center_ninety"/>


## Create Pipeline Asset 
This will add the URP package to your project. Afterwards, create a folder in the Assets folder called "Rendering". Inside it, create Rendering >> Universal Render Pipeline >> Pipeline Asset (Forward Renderer). 

<img src="https://www.dropbox.com/s/9rmnuz90dwah5pd/2.png?raw=1"  class="center_ninety"/>

## Modify Project Graphics Setting

Go to Edit >> Project Settings >> Graphics and select the UniversalRenderPipelineAsset you have created in the previous set as shown:

<img src="https://www.dropbox.com/s/ws2vqn922gxnqrb/3.png?raw=1"  class="center_ninety"/>

## Create 2D renderer 
Since we are working with 2D games now, we need to create a 2D renderer as our Universal Render Pipeline Asset's Renderer. Right click inside the Render folder and Create >> Rendering >> Universal Render Pipeline >> 2D Renderer as shown:

<img src="https://www.dropbox.com/s/p76bbmrb2ke6nzq/4.png?raw=1"  class="center_ninety"/>

Click on the UniversalRenderPipelineAsset you created before and load the 2D Renderer you just created under its Renderer List in the inspector and we are **done** with setting up URP for your project. 

<img src="https://www.dropbox.com/s/i54jrije3aylsx3/5.png?raw=1"  class="center_ninety"/>

Open Fighter.scene and notice that everything in the Game window is dark, although you can clearly see in the Scene that there are some stuffs in the world as shown below. If you can't see anything on the Scene either, toggle the light bulb icon to *disable Scene lighting* (for our view when prototyping, but not the camera's). 

<img src="https://www.dropbox.com/s/m4fwprxh8e2bznl/6.png?raw=1"  class="center_ninety"/>


*Time to add some Lights, Camera, Action!*
