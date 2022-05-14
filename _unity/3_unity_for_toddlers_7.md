---
title: Checkoff
permalink: /unity/unity_for_toddlers_7
key: unity-unity_for_toddlers_7
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
Your task for checkoff now is to apply glow on Mario's shirt. For example, the following is done by adding some point light on Mario, then creating another texture only on the poses where Mario is running (use Photoshop or any image editor), and applying a Bloom filter. Ignore the dust particle effect. It is not part of the checkoff, but simply just a nice visual effect to add.  

<img src="https://www.dropbox.com/s/7dgb03qd2h139zy/glowmario.gif?raw=1"  class="center_ninety"/>


Implement all the following in any way you deem fit, be it following our tutorial above or other creative avenues:
1. Breakable bricks (you are free to define what breakable means, that is if the debris has collisions with the ground, tiny animations, etc). 
2. Glowing mario shirt when he is running, or jumping, or doing anything.
3. Use particle system to create any interesting object, you must utilise **textured sheet animation**.  
4. Create your own simple graph shader for anything in the scene that changes at runtime (hence you need to use the `time` node). 
5. Implement parallax background effect

Here's a checkoff gif sample to follow if you don't want to think too much:

<img src="https://www.dropbox.com/s/2jr0r8smr669kkc/checkoff.gif?raw=1"  class="center_ninety"/>

What to do next?
{:.error}

We are almost equipped to recreate World 1-1 of Super Mario Bros, with a few enhanced effects. In the next Lab, we will implement a Game and State Manager,  an Object Pooler, and also an Audio Mixer. It will be mostly housekeeping and some C# stuff that hopefully will help you to arrange your code in a neater way. Now that you have understood Unity basics, it's time to clean up these loose ends. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0NDQ4NzY2MjIsLTE1ODMwMDI5MjQsMT
k3NDQ4MjU5OSwtODYxNDUxNjg4LDk2MDYxNTYzMV19
-->