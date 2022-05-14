---
title: Postprocessing
permalink: /unity/unity_for_toddlers_4
key: unity-unity_for_toddlers_4
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

One final thing to do is to enable *postprocessing*. Postprocessing effects are full-screen, and it can really change how the game looks very quickly. See this [manual](https://docs.unity3d.com/Manual/PostProcessingOverview.html) for more information. 

## Enabling postprocessing
In order to "see" the postprocessing effect, you need to enable it in your Main Camera. 
* Click on the Main Camera
* On the inspector, ensure that the Renderer is set as 2D Renderer, 
* And then tick the **Post Processing** property

## Adding Volume and Override
We need to tell Unity the context of which postprocessing should be applied. To do this, right click in the Hierarchy and create Volume >> Global Volume. Over at the inspector side, create new Profile. Then, Add Override >> Post Processing >> Bloom. Enable the bloom filter and set the Threshold and Intensity as shown:

<img src="https://www.dropbox.com/s/lbldzi19aamutxf/26.png?raw=1"  class="center_ninety"/>

### Bloom Filter
Bloom gives the illusion of an extremely bright light and is a great way to enhance add visual ambiance to your Scene. The two properties we set here are:
* Intensity: the strength of Bloom filter
* Threshold: Filters out pixels under this level of brightness

> If we did not apply the glowGraph shader to the player and instead to use regular Sprite-Unlit-Shader, then the necklace or emission effect **will not be bright enough** for the Bloom Filter to pick up. Now save your scene and observe the nice effect in all its glory. Feel free to adjust the Light gameobject on the player: intensity or color to indicate different "aura". 


### Other Post-Processing Effects
There's a ton of other effects which we will not cover here, but worth exploring if it is necessary for your project. Another commonly used effect is the Depth of Field (adding Bokeh), Motion Blur, Screen Space Reflection, etc. You can view the full list [here](https://docs.unity3d.com/Manual/PostProcessingOverview.html).  Remember to not overdo, but only invest your time in features that *matters* to your game. 

## HDR 
To add even more punch with the colors, we can enable HDR. Head to glowGraph shader and click on the Color properties. Under Node Settings, change its Mode into HDR. You can now adjust the intensity of the color manually from the **Material's inspector.** 

<img src="https://www.dropbox.com/s/3pottvy752dx45r/36.png?raw=1"  class="center_ninety"/>

The result really packs a  punch, with more "glow" effect. You can increase the Bloom filter's intensity to your liking. 

<img src="https://www.dropbox.com/s/heibltje8snosmr/hdr.gif?raw=1"  class="center_ninety"/>


## Other Simple Shader Samples: Blinking, Glow Outline
You can also use the shader to dynamically change the texture of the object at runtime, such as creating this blinking mushroom:
<img src="https://www.dropbox.com/s/1tkclhjq6xxs3ye/glow.gif?raw=1"  class="center_ninety"/>

*Note: download the mushroom assets yourself. There's a lot of free ones that can be found in the internet.* 

The shader graph used to render the Blinking mushroom is:

<img src="https://www.dropbox.com/s/rt5tul1arhkqnn9/38.png?raw=1"  class="center_ninety"/>

You need a secondary texture that contains only its white pixels. This can be done fairly quickly in Photoshop or the likes.

To create a "glow" outline, you can use the [step](https://docs.unity3d.com/Packages/com.unity.shadergraph@6.9/manual/Step-Node.html) function, although it might not always be obvious. You can enhance the glow using Bloom filter later on. The shader graph for the glowy orange mushroom is as such:

<img src="https://www.dropbox.com/s/u4hb880t95x5laf/orangemushshader.png?raw=1"  class="center_ninety"/>

Note that this method does not always work for all kinds of texture. The gif above showed that the outline of the orange mushroom isn't that clear. The reason lies in how the **step** function operates:
* If you read the documentation, shader **step** function returns 1 **if the `Edge` Input is greater than `In` Input**. 
* The `Edge` input is receives by the texture's alpha channel, which has a value between 0 to 1.
* This works because most images are usually "blurry" on the side. You might need to edit the sprite image first and blur the edges in order to give a more discernible outline using the step function. 
* Then ignore the alpha using the Combine node so that the final output does not have that blurry edge from the original texture:

<img src="https://www.dropbox.com/s/ulokm6fxzcczkzz/40.png?raw=1"  class="center_ninety"/>
