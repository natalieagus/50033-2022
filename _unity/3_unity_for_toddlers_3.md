---
title: Shaders
permalink: /unity/unity_for_toddlers_3
key: unity-unity_for_toddlers_3
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




Creating your own shader is no easy feat. It is one of the hardest thing to do in fact, but ShaderGraph in URP made it manageable. In this section, we are going to make **custom shaders**  (and post-processing effect) so that we can convert our character from looking like this: 

<img src="https://www.dropbox.com/s/r46zjruyqdv75n2/nofilter.gif?raw=1"  class="center_ninety"/>

 To this:

<img src="https://www.dropbox.com/s/a1zs8qg6bpjmlql/postprocessing.gif?raw=1"  class="center_ninety"/>

## Creating a Basic Shader Graph
Under Assets, create a folder called Shader. Inside it, right click Create >> Shader >> Universal Render Pipeline >> Sprite Unlit Shader Graph. 

Name it: `glowGraph`. Double click the graph and you should be presented with the following graph editor UI:

<img src="https://www.dropbox.com/s/dzthchmatvy2szs/13.png?raw=1"  class="center_ninety"/>

**The way we encode logic to the Shader is via visual scripting.** Right now we only have two **nodes**, called **Vertex** and **Fragment**. The Shader that we are about to create will dictate how we should color a particular *fragment* (think of it as a single pixel) of our gameObject in our Scene (if any).   

### Vertex Shader and Fragment Shader
The figure below illustrates a simplistic summary of graphics pipeline:
<img src="https://www.dropbox.com/s/m1wjd0w2luykn7y/pipeline.png?raw=1"  class="center_ninety"/>

In graphics pipeline, there are *two* types of shaders that are commonly customisable (well, actually *three*. The other one being geometry shader but this is not 50.017 so we are going to spare you from that), called v**ertex shader** and **fragment shader**. 

**Vertex Shader:** applied very early in the graphics pipeline, and its job is to **transform** each **vertex's** 3D position in *virtual space* (world space) to the **2D coordinate** at which it appears on the screen (as well as a **depth** value for the Z-buffer; so that the GPU knows what ought to be rendered in front of what). **Vertex shaders** can manipulate properties such as position, color and texture coordinates, but *cannot* create new **vertices**. A vertex is normally composed of *several* **attributes** (positions, normals, texture coordinates etc.).

**Fragment Shader:** applied later in the graphics pipeline, and it defines **RGBA** (red, green, blue, alpha) colors for each **pixel** being processed. It can be programmed to operate on a per pixel basis and takes into account *lighting* and *normal* mapping.

You can learn all about Shaders glory using various 2D and 3D rendering API, for example using OpenGL [here](https://learnopengl.com/Getting-started/Shaders), or using Vulkan [here](https://vulkan-tutorial.com/Drawing_a_triangle/Graphics_pipeline_basics/Shader_modules), or using Metal [here](https://www.raywenderlich.com/7475-metal-tutorial-getting-started). Writing shaders from scratch is one of the hardest thing to do in life, so we gotta be really grateful with the existence of visual scripting tools like URP which allow us to write shaders without having to be bothered with shading language syntaxes. 

> But this doesn't mean that you can forget your linear algebra and programming knowledge, or basic logic. 

Let's say we want to shade a simple solid color on the object's **fragment**, e.g: **red** color everywhere, not affected by lighting or anything. In GLSL (that's OpenGL's shading language), we need to specify the vertex shader as follows:

```glsl
#version 330 core  
layout (location = 0) in  vec3 aPos; // the position of the vertex is at position 0 of its attribute. So this layout qualifier basically says "this data in this buffer will be attribute 0"
out  vec4 vertexColor; // specify a color output to the fragment shader  

void main() 
{ 
	gl_Position = vec4(aPos, 1.0); // see how we directly give a vec3 to vec4's constructor 
	vertexColor = vec4(1.0, 0.0, 0.0, 1.0); // set the output variable to a pure red color 
}
``` 

and the fragment shader as follows:
```glsl
#version 330 core  
out  vec4 FragColor; 
in  vec4 vertexColor; // the input variable from the vertex shader (same name and same type) 
void main() { 
	FragColor = vertexColor; // set fragment color to be equivalent to vertex color, which is red
}
```
GLSL seems daunting at first, like a nightmarish distant cousin of C programming language. We can create the same logic (that is to shade every pixel where the object in question is in red in this example) using shader graph easily without caring about which part should be the vertex shader or fragment shader. 

## Using ShaderGraph in URP 
Open glowGraph (double click on it). We will create a **material** later using this custom shader glowGraph, and load it as the **Material property** of a gameObject's **Sprite Renderer** so we can observe the output. 

### Texture2D property with _MainTex Reference 
Firstly, to use it with Sprite Renderer, we need to create a **Texture2D** **property** (whatever you create on the left hand side, the "Blackboard" panel will be the Shader's property) as shown, naming its **reference** (*Reference* field, **not** *Name* field in the Graph Inspector) as `_MainTex` **exactly** as it is required by SpriteRenderer2D in Unity: 

<img src="https://www.dropbox.com/s/h44f9nr8ljt9xul/16.png?raw=1"  class="center_ninety"/>

> The reason we need `_MainTex` property is because a Sprite Renderer *by default* uses the texture supplied in the Sprite property but uses the Shader and other properties from the Material property. We can use information of the texture (color) in the Sprite property (the `_MainTex`) later on for computations in the Shader to perform more complex computation for the color of each fragment; although in this case we are completely ignoring the color of `_MainTex` since we overwrite everything to be red. 


### Adding Nodes 
We can create nodes to form a shader graph, where each node represents a *logic*. You can right click anywhere at the empty space in the glowGraph window to create new nodes. There's various nodes for all sorts of logic: vector manipulation (dot product, cross product, etc), time stepper, noise generator, sin graphs, etc. 

To shade a simple red color on each pixel representing the gameObject, do the following: 
1. Create a **Color** **node**. Right click anywhere at the graph window >> Create Node, type "Color", then select Basic >> Color. 
2. Change the Color node's  value to red color (click on the color bar and adjust) 
3. Link the output of the node to the input Base Color of the Fragment node as shown (click and drag): 

<img src="https://www.dropbox.com/s/7ok7w386zp1eqev/14a.png?raw=1"  class="center_ninety"/>


Click on the **Main Preview toggle** on the top right hand corner of the graph editor to see what the output should look like, and then press **Save Asset** on the top left hand corner.

As said before, we need to create a **Material** (name it glowMaterial for example) using this glowGraph shader and then we need to apply it on a gameObject so that we can see the effect of our Shader. In your project folder, highlight `glowGraph` and right click on it, then select Material. This automatically creates a material utilising your custom shader glowGraph. 
> Alternatively, you can create a generic Material and change the shader in the inspector to be Shader Graphs/glowGraph. 

Rename this material into glowGraphMaterial as shown in the screenshot below:

> Ignore the other shaders. We haven't reached there yet.

<img src="https://www.dropbox.com/s/mnhfjlt0ye6b6su/15.png?raw=1"  class="center_ninety"/>

Now create any 2D sprite >> Square gameObject and place it on the scene. Load glowGraphMaterial onto the Material property of its Sprite Renderer, and you should see a bright square box, unaffected by lighting:

<img src="https://www.dropbox.com/s/577ywozbawdkkqe/17.png?raw=1"  class="center_ninety"/>

> If you don't see it, make sure its position property in Transform has negative Z value, so that it is "in front" of the background which Z position is set at 0. Alternatively, you can also change the Sorting Layer property of the sprite renderer.

## Creating an *Actual*  Shader Graph

Since our shader graph name is `glowGraph`, let's modify it further so that it lives up to its name. In order to allow our character to ***glow***, we need to:
1. Brighten all blue parts (the necklace and the "water" looking skill effect) on the character sprite.
2. Add post-processing Bloom filter (later, not part of our shader). 

We will do (1) in this `glowGraph` shader. In order to brighten all the blue parts of the Player sprite, we need to create a matching sprite to the Player sprite where we have exclusively the traces of the emission, as shown:

<img src="https://www.dropbox.com/s/1is17yzzv0cbl94/18.png?raw=1"  class="center_ninety"/>

 To create the sprites on the left, you can easily use photoshop and trace each blue pixels, replacing it with white. What's important is that the two sprites are of the **same dimension**. 
> For now, both sprites are given to you, simply search the name Player and Player_Emission in your Assets folder. 

The idea is that we want to:
1. **Load** both as textures to the Shader
2. **Multiply** the emission map with some color, e.g: teal
3. **Add** the output of (2) to the original Player Sprite, so we have this:

<img src="https://www.dropbox.com/s/pvaoapxl1zb3c6a/21.png?raw=1"  class="center_ninety"/>


### ShaderGraph Properties
Open glowGraph and add **two** more properties (on top of the MainTex you have created earlier with reference _MainTex):
* Another Texture2D named SecondaryTexture
* Color 

Click on **MainTexture** property, and click on the Graph Inspector toggle to view its Node Settings. Make sure you enter the Player sprite set as its Default property. 

Click on the **SecondaryTexture** property, and load Player_Emission sprite as its Default property. Set its reference to be anything you want, for example: _SecondaryTexture.

Finally, click on **Color** property and set the Default property color value to be something teal. Here's a screenshot to clarify what we mean by changing the Node Setting:

<img src="https://www.dropbox.com/s/5zt2ax287rmrela/19.png?raw=1"  class="center_ninety"/>

To use these three properties as part of the Shader's computation, we need to create Shader nodes. 

### Sample Texture 2D
Right click and create **two** Sample Texture 2D nodes. This node as the name suggests, samples textures from either properties you set before: MainTexture or SecondaryTexture. 

Drag MainTexture and SecondaryTexture from the Blackboard graph pane on the left, and connect each to the Texture(T2) input of each Sample Texture 2D nodes, as shown:

<img src="https://www.dropbox.com/s/y9gflai3m53f933/20.png?raw=1"  class="center_ninety"/>

### Multiply Node

Then create a Multiply node, and drag the Color property into the editor. Connect them as shown here:

<img src="https://www.dropbox.com/s/c7xozdw25kj9w0g/22.png?raw=1"  class="center_ninety"/>

It should be pretty obvious to deduce what's happening: we are doing element-wise vector multiplication. Since the color of each point in the secondary texture is white and everything else is black (zero color value), multiplying it with another color results in that color nicely overtaking all white points in the texture. 

> It doesn't matter if you connect R, G, B, A, or RGBA output to the multiply A-input port since the emission texture is purely black and white. 

### Add Node

What's left now is to add the RGBA output of the main texture to the RGBA output of the previous step. Create an Add node and connect them accordingly. 

<img src="https://www.dropbox.com/s/8wagbjupbl9ghtg/23.png?raw=1"  class="center_ninety"/>


### Splitting and Combining Channel 

We cannot connect the output of Add node directly to Base Color input of Fragment node because of the mismatch in the dimension. Base Color in Fragment only accepts RGB, and the Alpha is treated as a separate input port.

Hence, we need to split the RGBA Channels, and Combine the back the RGB while keeping the Alpha channel separate. Create a Split and a Combine Node, and connect them as such. You should see in the Main Preview (right click >> quad to see it on a plane instead of a circle) that the Player sprite now has a nice teal color at the emission parts:

<img src="https://www.dropbox.com/s/oty18nmndzcozv7/24.png?raw=1"  class="center_ninety"/>

### Saving Asset and Loading the Material to Player
To test the shader, click Save Asset as usual in the graph editor, and load the material utilising this glowGraph shader (e.g: glowMaterial you created earlier) as the Material property of the SpriteRenderer component in the Player gameObject. You should see something like this, where the player's necklace is now brighter. 

<img src="https://www.dropbox.com/s/f550i2f52jyo74w/25.png?raw=1"  class="center_ninety"/> 

