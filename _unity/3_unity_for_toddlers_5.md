---
title: Parallax Background
permalink: /unity/unity_for_toddlers_5
key: unity-unity_for_toddlers_5
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

Another nice effect that you can add to your game is parallax scrolling. Parallax scrolling is a technique in computer graphics where background images move past the camera **more slowly** than foreground images, creating an **illusion** of **depth** in a 2D scene of distance.

Here's a preview of such effect. The foreground trees move with Mario but the other trees move *slower*, hence creating the illusion that the background trees are *further* away than the foreground trees. 

<img src="https://www.dropbox.com/s/df2j48pbjkggris/pbg.gif?raw=1"  class="center_ninety"/>

There's a lot of methods that can do this, such as creating different gameObject for each sprite, and then follow Mario at different speed, spawning new ones once the end of the sprite is about to be reached, like [this online tutorial](https://www.youtube.com/watch?v=3UO-1suMbNc):

<img src="https://www.dropbox.com/s/63oy3xt9a0l2bjh/pbgeg.gif?raw=1"  class="center_ninety"/>

However, we are going to take this opportunity to learn more about Cameras:
* Setting **Layers** and **Culling Mask**
* Setting Texture **offset** 
* Using **secondary cameras** and **combining** the output with the Main Camera

Our scene will look something like this. The parallax scrolling effect is achieved without having to spawn new GameObjects. 

<img src="https://www.dropbox.com/s/z0uq00ad77x1kri/prevpbg.gif?raw=1"  class="center_ninety"/>


Let's get started! 

## Getting Parallax Background Asset
The first step is to get vector images that support parallax scrolling. These backgrounds are mainly composed of different layers. You can download some samples for free online such as from [here](https://www.freepik.com/free-photos-vectors/parallax-background). The specific one that was used for this demo can be found [here](https://digitalmoons.itch.io/parallax-forest-background). 

Import them as sprites to your project, and change its **Texture** Wrap Mode property into **Repeat**. 


## Creating Materials and Layers for Each Background Object
Then create a Material for each sprite, with the following setting:
* Shader: Universal Render Pipeline/Unlit (only for this example where we don't need it to be affected by light)
* Surface Type: Transparent
* Blending Mode: Alpha 
* Base Map Surface Input: each parallax background Sprite.

An example is as shown:
<img src="https://www.dropbox.com/s/82pjm6nnj4699iz/27.png?raw=1"  class="center_ninety"/>

Then create a new GameObject and name it ParallaxBG. Create as many children 3D >> Plane gameObjects as the background sprite layers. Load each of the corresponding Material you just created above to each GameObject Mesh Renderer's Material property:

<img src="https://www.dropbox.com/s/x6zl7rxmfe2nrg6/28.png?raw=1"  class="center_ninety"/>

Don't worry if the order is messed up for now, we will fix that later with the cameras. Also, create a Layer for each gameObject, and match them accordingly. In the example above, BG0 GameObject is loaded with BG0 material, and has a **Layer** called BG0 as well. 

## Creating Secondary Cameras
Create a new Empty GameObject and name it ParallaxCameras. Under it, create as many Cameras as the number of sprites making up your Parallax Background. The idea is to allow each Camera to only "see" each corresponding background item, and then combine all their views together with the MainCamera to make up the final output. 

### Setting Camera Culling Mask and Priority
For each Camera GameObject, set its **Culling Mask** to correspond to each Layer so that it can only "see" the right gameObject. Then, set the **Priority** property of the Camera as well. **The object that's supposed to be rendered "in front" should have higher priority value**. We start with -2 for the foreground, -3 for the one right behind it, and so on, with -11 as the tenth layer. Here's a reference screenshot:

<img src="https://www.dropbox.com/s/q8ea7c6hirt4yfn/29a.png?raw=1"  class="center_ninety"/>

Now click on the Main Camera and ensure that the Culling Mask does NOT include any of the layers for your background gameObjects so the Main Camera *"does not see them"*. Also, set its Background Type to be "Uninitialized". Ensure that you disable any Background "wallpaper". The Main Camera's preview should be dark as shown: 

<img src="https://www.dropbox.com/s/gpu4qg1ijnk2tik/30a.png?raw=1"  class="center_ninety"/>

Note that the **Priority** property of the main camera is -1, which results in its output rendered **above** the output of the other cameras with **lower priority**. Your scene preview should look somewhat like the above now. 

## Scrolling the Background
Instead of physically moving the Background's transform, we will scroll the background by programmatically change each background object's Material **x offset** at runtime at different speed. Here's the effect from doing so:

<img src="https://www.dropbox.com/s/dyyclm0wvhr1gjl/offsettex.gif?raw=1"  class="center_ninety"/>

Create a new script and name it ParallaxScroller.cs, and declare the following variables and `Start` method:

```java
public  class ParallaxScroller : MonoBehaviour

{
	public  Renderer[] layers;
	public float[] speedMultiplier;
	private float previousXPositionMario;
	private float previousXPositionCamera;
	public Transform mario;
	public Transform mainCamera;
	private float[] offset;

	void  Start()
	{
		offset = new  float[layers.Length];
		for(int i = 0; i<  layers.Length; i++){
			offset[i] = 0.0f;	
		}
		previousXPositionMario = mario.transform.position.x;
		previousXPositionCamera = mainCamera.transform.position.x;
	}
}
```

The offset array keeps track of each background object's material x-offset value. Initially, they're all set as zero. We then compute how much mario has moved during `Update` and keep track of the MainCamera's location. This because sometimes at the **edges** of the map, the Main Camera doesn't move anymore and we do not want our Parallax Camera to move in this case. It will look weird.

Implement the `Update` function as such:

```java
void  Update()
{
	// if camera has moved
	if (Mathf.Abs(previousXPositionCamera  -  mainCamera.transform.position.x) >  0.001f){
		for(int i =  0; i<  layers.Length; i++){
			if (offset[i] >  1.0f  ||  offset[i] <  -1.0f)
				offset[i] =  0.0f; //reset offset
			float newOffset =  mario.transform.position.x  -  previousXPositionMario;
			offset[i] =  offset[i] +  newOffset  *  speedMultiplier[i];
			layers[i].material.mainTextureOffset  =  new  Vector2(offset[i], 0);
		}
	}
	//update previous pos
	previousXPositionMario  =  mario.transform.position.x;
	previousXPositionCamera  =  mainCamera.transform.position.x;
	}
}
```
Then load the script to the **parent** gameobject of all your background objects, and **set the parameters accordingly** in the **Inspector** depending on the number of parallax background layers you have. Enter any speed multiplier that you want for each layer, just ensure that they're in decreasing order so the background objects scroll a little slower than the others in front of it. 

<img src="https://www.dropbox.com/s/6p6lxbr3ga8jqmr/31.png?raw=1"  class="center_ninety"/>

Test run and we are done. You should have a nice parallax scrolling effect viewable from the Game window. 
