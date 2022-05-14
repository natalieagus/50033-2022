---
title: Checkoff
permalink: /unity/unity_for_newborns_3
key: unity-unity_for_newborns_3
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

Review our lab recording (the playlist for all recordings can be found in our course handout) to know the final state of the project before proceeding to implement the checkoff. The gif below also summarises the desired end state for this lab:

<img src="https://www.dropbox.com/s/05k8w5sa2qvm66o/checkoff1.gif?raw=1"  class="center_ninety"/>

Once you've implemented everything in this handout, then for **checkoff** you're required to implement a **restart** mechanism when game is **over**. You need to reset everything: score, player position, etc. 

*You’re free to implement it in any way*. *It will not affect your checkoff score.* The grading for this lab is **binary** (completed or not completed). 

***Read the course handout to find out the checkoff procedure for each lab.*** 

What to do next?
{:.error}

It's been hours but we are nowhere near a completed game (unless of course you have prior experience with Unity):

-   No **sound effect** or **animation** (lack of visual feedback)
-  No **platforms** implemented yet (it's a platformer game!)
-  There’s **no game manager** of any sort, and `score` is sloppily stored in `PlayerController.cs`
-   There’s no *centralised* way for keeping track of **states** (score, player state, etc)
-   The “Enemy” is kinda predictable or boring, and Mario's scoring system doesn't work that way. 
- ...etc

We will try to improve our game and learn some common C# coding practices in the next few parts.










<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwNDgzMzM4NjMsLTE5MTY4NDA2NjEsNT
I3Mjc2MzE1LC0yMDg1NjU1MjI5LC0xOTEzNzYyMjg5LDc2OTYx
NTEwNywxNDU5MjI0NzM1LDI0NjQwOTEwMCwtMTU0MzQ1NDc2MS
wxMjA2NTgzNTE1LC03NTcwNDk0NTcsLTg3ODE1OTIxNywtMzIx
OTkwNzU4LDIzMzU4NTE0MCwtMTAxODM3MzkxOSwtMTM5OTYxMT
M4MSwtMjA2MTM1NTc2M119
-->