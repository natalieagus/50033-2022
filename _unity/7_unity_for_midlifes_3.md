---
title: Traversing Between Navmesh Surfaces
permalink: /unity/unity_for_midlifes_3
key: unity-unity_for_midlifes_3
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

### OffMeshLinks 
Sometimes we need to *connect* between two or more NavMeshSurfaces, and we can do this using OffMeshLinks (Unity default) or NavMeshLinks. 

> From Unity's Official Documentation: OffMeshLink component allows you to incorporate navigation shortcuts which cannot be represented using a walkable surface. For example, jumping over a ditch or a fence, or opening a door before walking through it, can all be described as Off-mesh links.

Open the Scene inside NavMeshIntro >> Scene4. **Observe** the GameObject "Start" and "End" under Ground >> Links. These two indicate the location of "entry and exit" point between the green and yellow surfaces. Click on the Ground and realise that there's a component called `Off Mesh Link`, where Start and End properties contain references to the Start and End gameobjects:

<img src="https://www.dropbox.com/s/32yg2kmzecis6i4/7.png?raw=1"  class="center_ninety"/>

You can read more details about the rest of OffMeshLink's properties [here](https://docs.unity3d.com/Manual/class-OffMeshLink.html).

Now Floor1 (yellow) is placed slightly above Floor3. We can *automatically* generate OffMeshLink by enabling the Generate OffMeshLinks property on the Floor1 Mesh Renderer. If this property is not ticked, then the OffMeshLink generated is only the one between Floor1 and Floor2 which we added earlier. Click "Bake" under the Bake tab and you shall see the NavMeshSurface generated as follows:

<img src="https://www.dropbox.com/s/uz0mgum6agaar0e/8.png?raw=1"  class="center_ninety"/>

It is *possible* to auto-generate OffMeshLink as long as the "Drop Height" property inside the Bake tab is obeyed between Floor1 and Floor3:

<img src="https://www.dropbox.com/s/2t7rcxwl84plkhl/9.png?raw=1"  class="center_ninety"/>

You can also **animate** the Player gameobject when on OffMeshLink using `agent.isOnOffMeshLink` check. Open the script attached to the Player in Scene4, and read the following code:
```java
        if (agent.isOnOffMeshLink && gameObject.tag == "customplayer" && !isPlaying)
        {
            gameObject.GetComponent<Animation>().Play();
            isPlaying = true;
        }

        else if (!agent.isOnOffMeshLink && gameObject.tag == "customplayer" && isPlaying){
            if (isPlaying){
                isPlaying = false;
                gameObject.GetComponent<Animation>().Stop();
            }
        }
```
There's already an *animation* component attached on the Player gameobject (just a single animation, and we will explicitly play it from the script so there's no need for an Animator). We can easily check if the agent is on or off the OffMeshLink and we can play or stop the Animation accordingly. 

### NavMeshLinks
Another alternative is to use NavMeshLink (part of NavMeshComponents package we installed earlier). Neither is favoured over the other, and it depends on your usage. For example, in NavMeshLink, you can set the **link width**, whereas OffMeshLink is just one single path. 

Open the Scene in Scene3 folder, and click on the gameObject StairsLink. Notice that it has a component called NavMeshLink. Similar to OffMeshLink, you can define a start and end point on separate NavMeshSurface. You can also adjust the **width** of the link:
<img src="https://www.dropbox.com/s/g41stknj7cg3jpf/10.png?raw=1"  class="center_ninety"/>

More details about its other properties can be found [here](https://docs.unity3d.com/Manual/class-NavMeshLink.html). 

## NavMeshSurface

Finally, if we want to create a *walkable* surface on vertical walls, we need to add the NavMeshSurface Component. This feature comes from the NavMeshComponent asset we installed earlier (doesn't come with standard Unity installation). Standard Unity installation only allows us to create NavMesh out of meshes that are *aligned* with the **world** XZ plane, or more precisely: the XZ plane on the positive global Y-axis side. 

Open Scene4 (inside NavMeshIntro >> Scene4). Notice how we have NavMeshSurface component attached to WallRed and WallBlue. If we click *Bake*, this will generate a NavMesh for each wall, at its XZ plane on the positive Y-axis side:

<img src="https://www.dropbox.com/s/sy95yow2tssorfs/11.png?raw=1"  class="center_ninety"/>

We can add a couple of OffMeshLink to connect each surface, hence allowing the Agent on the Player gameobject to travel between the three surfaces. One thing to note is to align the *local* Y-axis orientation of the player to be the same as the surface it is on as shown:

<img src="https://www.dropbox.com/s/gsyb8bups964q44/wallkers.gif?raw=1"  class="center_ninety"/>

This can be done by enabling the `updateUpAxis` property:
```java
NavMeshAgent agent = gameObject.GetComponent<NavMeshAgent>();
agent.updateUpAxis = true;
```

# NavMesh for 2D Games
As mentioned previously, NavMesh works on *Mesh Renderer*, which means that it won't work by itself on regular Sprites or TileMap. This asset called [NavMeshPlus](https://github.com/h8man/NavMeshPlus) is a great workaround if you want to create NavMesh on 2D surfaces. You can add the package from Git URL: `https://github.com/h8man/NavMeshPlus.git#master`

> NavMeshPlus comes with NavMeshComponents, therefore to avoid **conflicts**, you may remove NavMeshComponents from the package registry. 

Then import the asset `navmesh2dintro` from the course handout.  to your project.

> NavMeshPlus comes with NavMeshComponents, and you may be faced with conflicts. To avoid **conflicts**, you may remove NavMeshComponents from the package registry. 


## Create a TileMap
Tilemap is another handy tool to create 2D maps, especially for top-down games which levels or world would otherwise be too cumbersome to be created as separate Sprites. You need to add the 2D Tilemap Editor from the **package manager.** Once downloaded, open the Scene Tilemap (inside NavMeshIntro >> TileMap) and click on the Grid gameobject. This gameobject was created using Create >> 2D >> Tilemap >> Rectangular. 

<img src="https://www.dropbox.com/s/pl2ndp6q8hd1dcq/12.png?raw=1"  class="center_ninety"/>

You'll have a Grid gameobject with one child gameobject for you to place your tilemaps. You can continue **adding** more Rectangular TileMap as a child of Grid. **Click** on Grid and then click on **Open Tile Palette** in the Scene. This allows you to create new **Palettes** which you can use to **paint** the Tilemap. You can drag **tile sprites** onto an empty palette. Once loaded, you can start using the Tile Sprites to pain the tilemaps. You can select the **Sorting Layer** and set the value of **Order in Layer** properties in the **TileMap Renderer** to determine which layer should be rendered on top. 

The gif below summarises the entire process:
<img src="https://www.dropbox.com/s/5yqmyxqklw83utl/tilemapcreate.gif?raw=1"  class="center_ninety"/>


## Create NavMeshSurface2D and NavMeshModifier
Click on the NavMesh gameobject, and notice there's two NavMeshSurface2D components placed on it:
- The first one monitors its "children", which is a collection of Sprites (**collect object** property set to Children)
- The second one monitors the "world" (**collect objects** set to All)

When you click "Bake", this generates Navmeshes on corresponding sprites or tilemap. Of course **NOT** all Sprites or Tilemap are considered into NavMesh baking by default. You'd need to **add** NavMeshModifier component to tell the system that "this object" should be considered during baking. You can even define whether it is Walkable or not Walkable as usual:

<img src="https://www.dropbox.com/s/zzm4w4t1u76kr7p/navmeshmodif.gif?raw=1"  class="center_ninety"/>


## Generate NavMeshSurface2D from Polygon 
You can also change the NavMeshSurface2D property: Use Geometry to use Physics Colliders instead so that we can generate NavMesh following a defined 2D collider instead of the Sprite Renderer. 

<img src="https://www.dropbox.com/s/glg9nu0qcpvj3uv/poligon.gif?raw=1"  class="center_ninety"/>


### NavMeshLink
Similar to NavMeshes in 3D, we can set NavMeshLinks as usual to connect two baked NavMeshSurface2Ds together:

<img src="https://www.dropbox.com/s/1l9e6awi5s2ptgc/13.png?raw=1"  class="center_ninety"/>

# NavMesh Data
If you haven't realised already, the data for NavMesh (2D or 3D, doesn't matter what it is) is stored typically at the directory where the Scene is, inside a folder with the same name as the Scene:

<img src="https://www.dropbox.com/s/xyayllmcs3h6rdn/14.png?raw=1"  class="center_ninety"/>

Here's the data for the Wallkers Scene:
<img src="https://www.dropbox.com/s/8f4qtddcslueddy/15.png?raw=1"  class="center_ninety"/>


# Finding "Click" Location in the World Coordinates
Notice how in both 2D and 3D Navmesh scene demos, we can always click to screen and the Player is able to navigate to that location. 

## 2D Click Location
In 2D setup with **Orthographic** camera, we need to transform the screen coordinate into the world coordinate using `Camera.main.ScreenToWorldPoint`. Open the script `NavmeshController.cs` and observe the following instructions:
```java
        if (Input.GetMouseButtonDown(0))
        {
            Vector3 mousePos = Input.mousePosition;
            mousePos.z = Camera.main.nearClipPlane;
            worldPosition = Camera.main.ScreenToWorldPoint(mousePos);
            // Debug.Log(worldPosition);
            agent.destination = new Vector3(worldPosition.x, worldPosition.y, zpos);
        }
```

## 3D Click Location
In 3D setup with **Perspective** camera, we use **raycasting** instead. That is, we will cast a Ray originated from the main camera towards the direction of mouse click on screen, and find if any the **Hit** location on **any 3D Physics Collider** in the Scene. This method wouldn't have worked in 2D games because *obviously* we **do not use 3D colliders** there, which belongs to the Physics Engine and not 2D Physics Engine.  

Open any PlayerController script used in Scene1 to Scene4, or Wallkers, and you should see the following code that computes the exact World location for the agent to go to on Mouse Click:
```java
        if (Input.GetMouseButtonDown(0))
        {
            Ray ray = cam.ScreenPointToRay(Input.mousePosition);
            RaycastHit hit;

            if (Physics.Raycast(ray, out hit))
            {
                agent.SetDestination(hit.point);
            }
        }
```

# Navigating Between Waypoints
We often need to define a specific path for bots to patrol, and we can do that by creating a few gameobjects as WayPoints, and then ask the Navigation Agents to patrol between the waypoints as such:

<img src="https://www.dropbox.com/s/af3qb6i6s429yyp/waypoints.gif?raw=1"  class="center_ninety"/>

This is done in a simple script `SwitchWP` placed inside the gameobjects WP0 and WP1 in Tilemap Scene:
```java
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class SwitchWP : MonoBehaviour
{
    public int destination;
    void OnTriggerEnter2D(Collider2D other)
    {
        Debug.Log("collided");
        if (other.tag == "Player")
        {
            other.GetComponent<NavmeshController>().goToWP(destination);
        }
    }
}
```
You can define a more complex patrol path using some kind of State Machine, which we will cover in the next tutorial. 


# Next
There's no checkoff with this lab. In the next lab, we will utilise our knowledge on NavMesh and Scriptable Object to create patrolling bots. 

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQ3ODc4NjcyMywyMDcxMDcwODg4LC0xND
c5ODE4NzYyLC0xNTIxMzg4NzM3LDk0NTIxNzMwOCwtNTM4OTcx
MDI2XX0=
-->