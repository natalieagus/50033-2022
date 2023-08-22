---
title: Unity for Newborns
permalink: /unity/unity_for_newborns
key: unity-unity_for_newborns
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

The contents of these labs are made to teach and stress some learning points and for mere “practice”, e.g: getting used to Unity layout, terminologies, etc.

By no means we claim that they are the best practice, in fact some of the ways may be convoluted and they won’t be done exactly this way by experienced coder but we can’t teach the “best practices” right away because it relies on many prior knowledge.

Experienced coders: keep in mind that you too were once a _noob_. You made mistakes.

_If you realise that some parts are troublesome or ineffective, then good for you. It means that you’re **smart** and **experienced**, and from now on you can embark on the journey to customize it to a more fitting way: simpler, better, more efficient, whatever it is. You can tell our teaching team your personal opinion, and constructive criticism is always welcome after class. We however expect a certain kind of mutual respect during the lab hours._

We do not give a **trivial** hand-holding step-by-step tutorial here. We do condense some steps. Remember that this is still part of your graded lab assignment, so you need to **imply** some parts based on standard programming knowledge as a CS student.
{:.error}

# Learning Objectives: 2D Basics

- **Unity editor basics**: layout, arrangements of project files
- Edit **scene**, add **GameObject** & elements, create **prefabs**
- C# **scripting** basics
- **Unity Life Cycle** introduction and callback functions: Update(), Start(), OnTriggerEnter(), etc.
- Physics **simulation** basics: RigidBody2D, Collider2D
- **Binding** keys for input
- **UI elements**: Canvas, Text, Button
- Basic experience on **events**: OnClick() for Button

# The Classic Super Mario Game

<img src="https://www.dropbox.com/s/ym7mp79vgc7hpcf/mario.png?raw=1"  class="center_ninety"/>

The goal of this simple 4-week lab is to recreate basics this classic platform game: **Mario**, step by step and complete it by the end of Week 4. We will try to rebuild World 1-1 as closely as possible, although due to constraints of time, some features may be omitted. In Week 5 & 6, we upgrade our skills to explore Unity3D.

# Preparation

**Download the starter asset** from your **course handout**, under "Class Calendar" heading, _Week 1 Lab Session row._ This is a starter asset that you can import to your project and complete the lab. It contains all required images, sound files, etc so we can save time searching all these stuffs.

**_We do not own any of the assets and are simply using them for non-profit educational purposes._**

_Note for experts in Unity:_ If you're already an expert in Unity, you're welcome to implement the final state of the project in any way you want, and implement the Checkoff. Jump to the [Checkoff heading](https://natalieagus.github.io/50033/2021/01/01/unityfornewborns.html#checkoff) right away for more information.

## Create New Project

Before we begin, create a **new unity project**.

<img src="https://www.dropbox.com/s/90e1wy14v2nabp0/1.png?raw=1"  class="center_ninety"/>

Then, import the asset you downloaded. You should see a list of assets on the Project tab in the Unity editor.

It is also crucial to have a good code editor. Add your own script editor as shown. _Visual Studio Code is recommended._

<img src="https://www.dropbox.com/s/nzxmhogb9wfjfv7/2.png?raw=1"  class="center_ninety"/>


See the guide : [https://code.visualstudio.com/docs/other/unity](https://code.visualstudio.com/docs/other/unity). For Mac users, simply: `brew install mono` on top of what the guide above told you to do. If you are NOT familiar with installing SDK or frameworks or editing `.json` files then just use the easy **MonoDevelop IDE** instead (heavier, not as pretty, but saves you the hassle).

Finally, check if `dotnet` is installed:

<img src="https://www.dropbox.com/s/anhmyp4nlqe88c1/3.png?raw=1"  class="center_ninety"/>

If you're using VSCode, edit `settings.json` to include the following:

```java
"omnisharp.defaultLaunchSolution": "latest",
"omnisharp.path": "latest",
"omnisharp.monoPath": "",
"omnisharp.useGlobalMono" : "always"
```

<img src="https://www.dropbox.com/s/btqqvzs554xcw3f/4.png?raw=1"  class="center_ninety"/>

After installing `dotnet` and `mono`, then go to Unity > Preferences > External Tools > and click **regenerate project files**:

<img src="/50033/assets/images/lab1/csproj.png"  class="center_seventy"/>

Debug:
{:.error}

- For macOS user, use "omnisharp.monoPath": "/Library/Frameworks/Mono.framework/Versions/Current" if the above does not work.
- You can read the output log of omnisharp (**View > Output **then go to **OmniSharp Log**), and if it omplains about something like [.Net Framework 4.7.1 version](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net471) not found, then just install that version.

## Housekeeping

To work better, we need to set up the UI in a more comfortable way. We need at least the following windows:

1.  Inspector (to have an overview of all elements in a GameObject),
2.  Game Hierarchy (to have an overview of all GameObjects in the scene)
3.  Scene Editor (to add GameObjects and place them)
4.  Project (to show all the files),
5.  Console (to see printed output)
6.  Game Screen (to test your game)

Go to “Window” and select the windows that you want. Explore the options and arrange the tabs in any way you like.

Then, go to **Scenes** folder and rename your scene in a way that it makes more sense than “_SampleScene_”. Create **two more folders** called “**Scripts**”, and “**Prefabs**” under "**Assets**". We will learn about them soon.

This is how my layout looks like.

<img src="https://www.dropbox.com/s/7w0bgxjwlifxj7n/5.png?raw=1"  class="center_ninety"/>

# The Basics

## Add GameObjects to the Scene

Everything you see on the game scene is a **GameObject**. It is the [base class](https://docs.unity3d.com/ScriptReference/GameObject.html) of all entities in the Unity Scene.

**Let’s add Mario to the scene.** Right click in the Hierarchy tab and create a 2D Object with **Sprite** component. Change its name to “Mario”.

<img src="https://www.dropbox.com/s/oe5u0f2cekg5syw/6.png?raw=1"  class="center_ninety"/>

## The Inspector

On the **Inspector**, head to the `SpriteRenderer` element and add the _mario_idle_ sprite. You should see something like this in the end. The inspector is pretty straightforward. It contains all elements attached to a particular gameobject:

- `Transform` dictates where this GameObject is in the Scene. You can change its coordinates with respect to the Camera Object so you can view it from differnt angles.
- `SpriteRenderer` dictates what the object should look like on the Scene.

We can attach more elements to the GameObject, such as `RigidBody` for physics simulation. More on that later.

<img src="https://www.dropbox.com/s/plj2iw35xcmlurh/7.png?raw=1"  class="center_ninety"/>

## The Camera

The “Scene” is the entire game world. What users can “observe” depends on the main camera. Notice that there’s always a Main Camera GameObject in your scene.

In your “Game” tab, you may see Mario with a blue background, but in the Scene tab, its Mario with a grey background. That’s due to your **camera setting.** We can adjust the camera to give different POV to the user or background color.

<img src="https://www.dropbox.com/s/8dueft8adz69m17/8.png?raw=1"  class="center_ninety"/>

It is worth noting that you might want to set the **game aspect ratio**.

- Click the drag-down menu as shown and select the option that you’re most comfortable with.
- E.g: selecting “Free Aspect” means that your window size will affect the camera “view”.

<img src="https://www.dropbox.com/s/pwvkuq6l4eok3ls/9.png?raw=1"  class="center_ninety"/>

## Setting up Inputs

Go to **Edit >> Project Settings** and click on **Input Manager**.

We want to test if we can control the movement of Mario using the keys “a” and “d” for movement to the left and right respectively.

- Check if the setting of “horizontal” axis is true. You can add your own key bindings here and name it.
- Later in the script, you can decide what to do if a certain named key is pressed.

<img src="https://www.dropbox.com/s/fi6f9dpdn0tt4pt/10.png?raw=1"  class="center_ninety"/>

## Creating a Script

Right click inside the **Scripts** folder in the **Project** window, create a new C# script `PlayerController.cs`. Here we will programmatically control Mario. Open the script with an editor of your choice.

**Let's attempt to move Mario via the script.** Firstly, Add `Rigidbody2D` component in the **Inspector** and set:

- `Gravity Scale` to `0`,
- `Linear Drag` to `3`
- `BodyType` to `Dynamic`

You are free to play with other parameters.

We can then control this component from the script. Paste the following inside `PlayerController.cs`:

```java
public float speed;
private Rigidbody2D marioBody;
// Start is called before the first frame update
void  Start()
{
	// Set to be 30 FPS
	Application.targetFrameRate =  30;
	marioBody = GetComponent<Rigidbody2D>();
}
```

Add the script to Mario (Add Component >> `PlayerController` script) then set `speed` to `70` in the Inspector. Afterwards, implement the **event callback** `FixedUpdate` as shown:

```java
void  FixedUpdate()
{
	float moveHorizontal = Input.GetAxis("Horizontal");
	Vector2 movement = new Vector2(moveHorizontal, 0);
	marioBody.AddForce(movement * speed);
}
```

You can **test run** that now Mario can be moved to the left and to the right using the keys “a” and “d” respectively. _However it doesn’t feel quite right. We will fix this later but first, lets learn about Unity event functions._

## Unity Order Execution of Event Functions.

We can implement the event functions in the script that’s attached to a particular GameObject. Notice that in the script we created,

- It inherits from MonoBehaviour, the base class from which every _Unity script_ derives. It offers some life cycle functions that makes it easier for us to manage our game.
- It comes with **two** starting functions for you to implement if you want: `Start()` and `Update()`.

`Start()` is always called once in the beginning when the GameObject is instantiated, and then `Update()` is called per frame. **This is where you want to implement your game logic.** The diagram below shows the order of execution of event functions. They run on a single Unity main thread.

<img src="https://docs.unity3d.com/uploads/Main/monobehaviour_flowchart.svg"  class="center_ninety"/>
    
The manual for all event functions can be found [here](https://docs.unity3d.com/Manual/ExecutionOrder.html). It is crucial for you to read it when the need arises, so you dont put weird things like implementing regular Physics simulation under `OnDestroy`.

Usually we don’t touch all of them. One of the more common ones to implement are: `Start, Update, FixedUpdate, LateUpdate, OnTrigger, OnCollision, OnMouse OnDestroy`, and some internal animation state machines if you use `AnimationControllers`. We will learn that in the next series.

## RigidBody2D Setting

Now back to the issue how Mario's movement doesn't seem quite right. He seems to be _sliding_. We would expect him to stop the moment we lift the key, wouldn’t we?

Setting the `BodyType` to `Dynamic` allows the **physics engine to simulate** forces, collisions, etc on the body. Since we’re adding Force to Mario’s body, it will obviously “glide” until the drag forces it to stop.

> Setting BodyType to `Kinematic` allows movement unders simulation but under very specific user control, that is you want to compute its behavior yourself -- simulating Physics under your own rule instead of relying on Unity's Physics engine. Read more the documentation [here](https://docs.unity3d.com/Manual/class-Rigidbody2D.html).
