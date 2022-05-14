---
title: Basics of Navigation System
permalink: /unity/unity_for_midlifes_2
key: unity-unity_for_midlifes_2
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

## NavMesh Generation
To allow automatic pathfinding, you need to first generate (*bake*) a **NavMesh** from all objects in your scene. 
> From Unity Documentation:  **NavMesh**  (short for Navigation Mesh) is a data structure which describes the walkable surfaces of the game world and allows to find path from one walkable location to another in the game world. The data structure is built, or baked, automatically from your level geometry.

Unity's built in NavMesh will **automatically** detect all 3D objects (mesh) that lie on the **XZ** plane. 

> We can create NavMesh on surfaces lying on other planes, such as the common XY plane in 2D games using `NavMeshComponents's Surface` (later). 

We need to also use **3D meshes** (mesh renderer) in order for Unity to *bake* the NavMesh, so 2D colliders can't be used to define "*walkable areas*". 
> There's plenty of workarounds that you can find [online](https://github.com/h8man/NavMeshPlus/wiki/HOW-TO), or [paid](https://assetstore.unity.com/packages/tools/ai/navigation2d-pathfinding-for-2d-games-35803#reviews) assets to help you. 

The rest of this tutorial will just introduce you how to use standard 3D Navigation System using Mesh Renderer (as per Unity base implementation). The 2D workarounds are built upon this knowledge.

### Import Asset

Import the asset from the Course Handout, then **open** the folder NavMeshIntro >> Scene1 and open the Scene Scene1. 

### Add the Navigation Window
**Add** the Navigation Window from Window >> AI >> Navigation. There's four tabs in the Navigation Window:
- **Agents:** You can define NavMesh Agents properties such as its step height (agents cannot "go over" steps more than this height), and max slope (agents cannot climb anything steeper than this slope)
- **Areas**: Similar to "layers", you can name different areas and its cost (second column). The cost will be used for pathfinding computation later on. Areas with lower cost is always more preferable. More about Navigation Area and cost [here](https://docs.unity3d.com/Manual/nav-AreasAndCosts.html). 
- **Bake**: This is where you generate the NavMesh with the respective agent size. You need to bake different NavMesh per agent. 
- **Object**: You can define the properties about the Object you clicked on the Scene:
	- Navigation Static: means this object will be used to compute NavMesh 
	- Navigation Area: set the area for this object. 

### Begin Baking
**Click** on the Ground GameObject on the left, and **go** to the Object Tab on the Navigation Tab. **Tick** the Navigation Static and **set** its Navigation Area into "Walkable". This turns the mesh of the Ground into something that will be considered for baking NavMesh. 

Then, **click** on the Bake tab and click "*Bake*". You should see the following in your Scene. That blue overlay is the baked NavMesh. During runtime, any NavMesh agent will use it to traverse the area. Note that anything that lies on the XZ plan is **automatically detected** as *bake-able area*.

<img src="https://www.dropbox.com/s/k95fevcj9f399sh/3.png?raw=1"  class="center_ninety"/>


Now click on the Player GameObject and notice that there's already a Navigation Agent component added to it, which means it can interact on the baked NavMesh. Ensure that the Player GameObject is nicely placed above the Ground NavMesh. 

<img src="https://www.dropbox.com/s/tp8586t8yzwp6k9/1.png?raw=1"  class="center_ninety"/>

The NavMesh you just baked matches the Agent's specification. If you were to create new agent type in the Navigation >> Agent Tab, then you can't use that new agent in this NavMesh baked for "Humanoid". We will get to other agent creation later on. 

Test run and click anywhere on the Game screen. The "player cylinder" will go to the place on the Screen that you click. 
 <img src="https://www.dropbox.com/s/qe039axo0y6jzlg/navmeshmove.gif?raw=1"  class="center_ninety"/>


## NavMesh Obstacle
As you have noticed, the NavMesh map so far is generated *before* the program is run, meaning that it is a **static** map. We can also choose to *create* obstacles at runtime, hence modifying the map as the program runs instead of only once in the beginning. To do this, we need the component **NavMesh Obstacle**. 

Open the scene inside NavMeshIntro >> Scene2 folder and click on one of the Obstacles gameobject:

<img src="https://www.dropbox.com/s/lue2ivt8b4uwrd7/4.png?raw=1"  class="center_ninety"/>

Observe the component NavmeshObstacle that can be added to any gameobject with a MeshRenderer. The property **carve** is ticked, which means that if you move this obstacle at runtime, the Navmesh will be updated accordingly. 

<img src="https://www.dropbox.com/s/vxwkmu1t89u41xw/obstacle.gif?raw=1"  class="center_ninety"/>

You can also *generate* NavMesh Obstacle at runtime. You'll need a reference to the gameobject with a MeshRenderer (as an obstacle to generate), and the NavMeshSurface that you want to instantiate this obstacle on. 

Open the Script `Generate.cs` inside the Ground gameobject in Scene2, and look at the following method:
```java
    void GenerateWall()
    {
        float x = (Random.value - 0.5f) * 20.0f;
        float z = (Random.value - 0.5f) * 20.0f;
        Vector3 pos = new Vector3(x, 1f, z);
        GameObject instance = Instantiate(wall, pos, Quaternion.identity, transform);
        instance.AddComponent(typeof(NavMeshObstacle));
        instance.GetComponent<NavMeshObstacle>().carving = true;
    }
```

To make any newly instantiated object an **obstacle** on the NavMeshSurface,  we simply need to just add the NavMeshObstacle component to the game object, and enable `carving`. We can create a prefab with this specification so that we don't have to have boilerplate code. 

**If we want to remove NavMeshObstacle at runtime**, we simply just **destroy** or **disable** the obstacle gameobjects. However, if we remove `static` obstacle like this Obstacle(4) during runtime, we are left with a "hole" in our NavMeshSurface:

<img src="https://www.dropbox.com/s/xqpmg86pyjwdhhv/5.png?raw=1"  class="center_ninety"/>

<img src="https://www.dropbox.com/s/f5did6ur3pem6qp/hole.gif?raw=1"  class="center_ninety"/>


If you want to remove static obstacles, you need to erase the existing data and rebuild the NavMesh. Of course by definition, static means **STATIC**, and you should use **OBSTACLE** if you’d like to modify stuff at runtime.


Although we are not supposed to change static (baked) objects at runtime, of course theres *workaround*. To remove static "not walkable" areas at runtime, we need to Destroy or disable that gameobject, delete the NavMeshSurface and then  rebuild our NavMeshSurface using the following Coroutine in the script: 

```java
   IEnumerator resetNavmesh()
   {
       while (staticObject != null) yield return new WaitForSeconds(1);
       surface.RemoveData();
       surface.BuildNavMesh();
   }
```

Notice that weird check on that `staticObject` reference. This is because you don’t really know when the `staticObject` is actually deallocated when you call `Destroy(staticObject)`. You can use `DestroyImmediate` but it is not recommended for build. Therefore you either need to **skip a frame** or you need to force wait for a little bit using a Coroutine before removing the existing NavMeshSurface data and rebuilding it. The same applies for `SetActive(false)`, the effect will only be observed in the **next frame** although the object is already flagged as inactive the moment you call the `SetActive(false)` instruction.

> If you feel that this is too hacky, that's because it is. It is probably best not to modify Navigation Static meshes during runtime. 
