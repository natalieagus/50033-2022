---
title: Adding 2D Lights
permalink: /unity/unity_for_toddlers_2
key: unity-unity_for_toddlers_2
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



The fun thing about enabling URP is that you can now lit up your sprites, thus creating a bit of a *mood*. Right click on your project hierarchy and observe that there are a few 2D Lights GameObjects that you can create, namely FreeformLight2D, SpriteLight2D, GlobalLight2D, and SpotLight2D.

### GlobalLight2D
Open Fighter.scene. Create a GlobalLight2D Game Object, and set the properties:
* Intensity of 0.4
* Target Sorting Layers: All (this determines which Sprite layers to lit up)
* Position at `(0, 0, -1)`

Toggle enable lighting at your Scene view and you should observe a somewhat dimly lit environment like this:

<img src="https://www.dropbox.com/s/na38excwcuqd9yf/7.png?raw=1"  class="center_ninety"/>

If you disable lighting, notice that everything in the Game window is dark, although you can clearly see in the Scene that there are some stuffs in the world as shown below. If you can't see anything on the Scene either, toggle the light bulb icon to *disable Scene lighting* (for our view when prototyping, but not the camera's). 

<img src="https://www.dropbox.com/s/m4fwprxh8e2bznl/6.png?raw=1"  class="center_ninety"/>

### SpotLight2D
Another type of light that you can add to the game os SpotLight2D. Create a gameObject of SpotLight2D type and place it as a child gameObject of the player. Place it somewhere near the Player's head to brighten up that area. You should see something like this in your Scene:

<img src="https://www.dropbox.com/s/bi163w5okkvbbqn/37.png?raw=1"  class="center_ninety"/>

Head over to the Light2D inspector and you observe that you can adjust its properties such as light **color**, **intensity**, **radius**, and **target sorting layers** (yes, you can ask light to just lit up certain areas and not the others). 

Expand the **blending** section and observe a few settings: this dictates how two or more light source should blend when they're near one another. Overlap operation of **additive** will cause the overlapping region to be very bright, while **alpha blend** allows the light with higher order to be rendered on after of the ones with lower order. The gif below demonstrates at first the blue light to be rendered after the green light, giving an overall blue-ish appearance. Afterwards, the light order is swapped so you see that the overall overlapping region has a green-ish appearance. Both lights are set to be "Alpha Blend". 

> Wait for a few seconds, the gif is quite long. 

<img src="https://www.dropbox.com/s/1xkizaxktv4y4kq/blending.gif?raw=1"  class="center_ninety"/>

## URP SpriteLit and SpriteUnlit Material 

The reason that Unity knows which Sprite should be affected by lighting (or not) is because of the material of the Sprite. To prove this point, expand the Environment GameObjects in the Hierarchy, and click on any background object. As an example, let's use `bg-2(1)`. 

Then in the inspector of `bg-2(1)`, change its Material into **Sprite-Unlit-Default**. You can see that part of the background corresponding to `bg-2(1)` isn't affected by the global light, and will just show its regular texture:

<img src="https://www.dropbox.com/s/lb5cr4jgl10ivgx/8.png?raw=1"  class="center_ninety"/>

The other background objects are affected by light because its material is set to **Sprite-Lit-Default**. Both **Sprite-Unlit-Default** and **Sprite-Lit-Default** are **Materials** that utilizes URP's **Shaders**. 

## Textures, Shaders, and Materials

Materials, Shaders, and Textures are three different things. We need the first two to render the Camera's view, and sometimes Textures as well if we do not want the object to simply contain solid colors. 

When we import images (.png, .jpeg, etc), we are creating **Textures**, also known as *bitmap images*.  Textures alone are not enough to dictate how Sprites or 3D Meshes should be rendered. 

We need **materials** for this. **Materials** are definitions of *how* a surface should be rendered, including references to textures used, tiling information, colour tints, *normal maps*, mask, and more. 

On the other hand, **Shaders** are small scripts that contain the **mathematical calculations** and algorithms for **calculating the colour of each pixel** rendered, based on the lighting input and the Material configuration. Therefore, it should be obvious that when we create materials, we need to specify the **Shader**. 

Let's create a *new Material* and demonstrate this knowledge. 
* Under Materials folder in your Assets, right click Create >> Material and name it GroundMaterial. 

* Set its shader to be Universal Render Pipeline/2D/Sprite-Lit-Default. Then, click on GameObject `tileset_15` and then, 
* Load this GroundMaterial under the Sprite Renderer component of `tileset_15`: the Material property. 

You will see that since GroundMaterial utilises the same shader as Sprite-Lit-Default material that comes with URP package, **there's no change in how the ground tiles look**. 

## Normal Mapping

When creating GroundMaterial, you might've seen a few properties that you can set, namely Normal Map, Mask, and Diffuse. We will briefly touch about Normal Map here. 

<img src="https://www.dropbox.com/s/pmxsenaj2duzfik/9.png?raw=1"  class="center_ninety"/>

In Computer Graphics, normal mapping is a texture mapping technique used for faking the lighting of **bumps and dents**. A normal map contains normal vector values at each point of the texture, encoded into the RGB color, so you will commonly see them in blue-ish hue as such: 

<img src="https://www.dropbox.com/s/yii9md00nu98wkp/NormalMap.png?raw=1"  class="center_ninety"/>

The blue hue comes from the fact that the majority of the surface is flat, and therefore the normal vector is {0,0,1} (full Z-direction, where Z-direction with respect to the local frame of the surface refers to the perpendicular vector (the surface itself is considered to be on X-Y plane *local* to itself). Mapping this into RGB: {0,0,1} results in the color blue. 

### Normal Mapping Example
Take for example this Mario brick + normal map applied (left) vs regular Mario brick (right). Two kinds of normal maps were applied: the one that matches the brick surface (the normal map above), and another is a rough rock normal map (so it's more obvious).

<img src="https://www.dropbox.com/s/4o9xbkb6ei9vh67/normalmap.gif?raw=1"  class="center_ninety"/>


### Creating Normal Maps
Now let's apply normal map to our floor. Our floor looks flat currently, and we need to generate normal maps from the floor's tileset (can be found under GothicVania Church folder >> Environment). 

Let's use an online tool to generate the normal map. This online tool: https://cpetry.github.io/NormalMap-Online/ is absolutely awesome (otherwise you can use Photoshop to generate normal map from a given texture too). Load the tileset image and you should be able to view and download a normal map for it as such:

<img src="https://www.dropbox.com/s/kb2gfwoz9d0w5kg/10.png?raw=1"  class="center_ninety"/>

Store the tileset normal map as an asset inside your Unity Project, and **set its Texture Type to be "Normal map"**. There's two ways to apply this normal map to our floor. We can apply it directly on the tileset Texture as a secondary texture, OR create a new material and specifically load this normal map into the shader. The latter is simpler, so we will use that. 

Open the GroundMaterial you have created earlier, and **load** the tileset normal map under the **Normal Map property**:

<img src="https://www.dropbox.com/s/ymy5lut1ulu6hpq/11.png?raw=1"  class="center_ninety"/>

**Load `GroundMaterial`** as the Sprite Renderer's Material property on `tileset_15` GameObject. 

To view the Normal Map, we need a light source. Create a new **SpotLight2D** with **Normal Maps** property enabled, and set its distance into something small as shown:

<img src="https://www.dropbox.com/s/hp0s58a2aab2kja/12.png?raw=1"  class="center_ninety"/>

Move the light around and you should be able to see the bumpy floor with normal map applied on it. 
