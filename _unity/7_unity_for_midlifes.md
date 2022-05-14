---
title: Unity for Midlifes
permalink: /unity/unity_for_midlifes
key: unity-unity_for_midlifes
layout: article
nav_key: unitypro
sidebar:
  nav: unitypro
license: false
aside:
  toc: true
show_edit_on_github: false
show_date: false
---

**Learning Objectives:** Pathfinding Using Navmesh
- **NavMesh Basics:**
	- Creating a click-to-move game
	- Connecting navmeshes together
	- Trigger animation using Offmeshlink 
	- Determining area and costs
- Using **NavmeshComponents**
	- Install navmeshcomponents package
	- Creating Navmeshlinks
	- Creating Navmesh agents and baking Navmesh surfaces
	- Orient agents on walls
	- Generating Navmesh obstacles at runtime
- Using **NavMeshPlus** to create NavMeshSurface on 2D Sprites or TileMap

# Installation

To begin, you need to install two things: NavMeshPlus and NavMeshComponents. Let's install the latter one first since it's easier. 
{:.error}

Navigation System is a part of Unity.AI module, and is extremely useful to perform pathfinding  in pre-determined (baked) maps automatically. To learn about this, **create a new 3D project** in Unity, and **import** the `navmesh3dintro` asset given in the course handout. This asset requires the `NavMeshComponents` package. Unity already has built-in Navigation System but this package has additional functionalities that will make your life way easier. 

Go to Window >> Package Manager as usual, and click *add package from git URL*:

<img src="https://www.dropbox.com/s/9fxkdkda52uhw4r/Screenshot%202021-06-22%20at%2021.24.13.png?raw=1"  class="center_ninety"/>

You can then paste the following URL for NavMeshComponents only: `https://github.com/Unity-Technologies/NavMeshComponents.git#package`
When the import is done, there should be no more error messages in the Console. The manual for this package can be found [here](https://github.com/Unity-Technologies/NavMeshComponents/tree/package). 

