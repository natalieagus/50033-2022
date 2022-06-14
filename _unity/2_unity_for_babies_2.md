---
title: Unity Physics 2D
permalink: /unity/unity_for_babies_2
key: unity-unity_for_babies_2
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

Now that our Mario can move around smoothly with proper animations, it's time we add some platforms. Adding platforms that Mario can jump on is easy: simply create a 2D object with Sprite Renderer and Box Collider 2D element on it:

<img src="https://www.dropbox.com/s/idppnefnujk71tj/13.png?raw=1"  class="center_ninety"/>

Make that brick into a prefab so that you can have a master copy because we are going to spawn many of these in the scene. 

> You might notice that Mario's animation doesn't get reset to "idle" state after he jumps onto the brick, because it is now colliding with a brick and not ground. Fix `PlayerController.cs` to consider the case where he can jump onto obstacles like this as well. 
>
>A cheap and easy way will be to add a **new** Tag to the brick, e.g: `Obstacles` and add that check as well when resetting the `onGround` animator parameter. 

## Effector2D
Suppose you want to create a platform that allows only one-way collision. You can upgrade your 2DColliders to be used with a new component called **[effectors](https://docs.unity3d.com/Manual/Effectors2D.html)**. 

To demonstrate how this works, let's create a platform where the character can "jump in" from underneath it but can stay upright on top of it. 
* Create a new 2D GameObject >> Sprite >> Square and name it PlatformThin. 
* Change its sprite to `tiles_187`
* Add BoxCollider2D component and edit its Collider to match the thin sprite.
	* **Tick** the Used By Effector property
* Now, add PlatformEffector2D component
	* **Tick** Use One Way
* Make PlatformThin a prefab, and spawn a two more in your scene, making a long edge

You should have something like this now:

<img src="https://www.dropbox.com/s/p3kfnt7dojisu2k/14.png?raw=1"  class="center_ninety"/>

Try jumping onto the brick and onto the platform **from right underneath the platform**. You should notice that Mario can't jump onto the brick from underneath it, while he can do so on the platform. 

 You shall experiment with other effectors and their properties as well so that you know what features are supported or suitable for your game idea: some kind of boost, buoyancy, etc. This is what you can use if you try to implement a pinball game with many different areas that can affect the kinematics of the ball . 

## PhysicsJoint2D 
Another interesting component of Unity's physics engine is the [joints](https://docs.unity3d.com/ScriptReference/Joint2D.html). It will safe you so much time if you want to implement any basic joints: hinge, spring, slider, wheel, etc. For example, the following is possible due to **SpringJoint2D** applied:

<img src="https://www.dropbox.com/s/zud51vgb1ofiux1/bouncehit.gif?raw=1"  class="center_ninety"/>

Follow these steps to create that Springy Question-box:
1. Create an **empty** GameObject with **two children** GameObjects. Name them:
	* **HittableSimple**
		* **TopCollider**
		* **EdgeDetector**
		
2. For the `EdgeDetector`, add **four** components: 
	* SpriteRenderer, with `tile_questionblock_0` as its Sprite
	* Rigidbody2D: with `Mass=0.0001`, zero Linear and Angular Drag, and Constraints to Freeze `X` position, as well as `Z` rotation. 
	* EdgeCollider2D: **edit the Collider** to align nicely with the bottom edge of the sprite 
	* SpringJoint2D (will edit its properties later)
	
3. For the `TopCollider`, add **two** components:
	* Rigidbody2D: with Body Type of `static` (we don't want it to be affected by Physics, we will only use it as an anchor for the SpringJoint in EdgeDetector GameObject). 
	* BoxCollider2D: edit the Collider to align nicely with the question box sprite

### SpringJoint2D
Now time to configure the SpringJoint2D inside EdgeDetector GameObject. It has a few properties, but we mainly are interested in setting up the Spring's anchor. If you're not familiar with how Spring works, you basically need to endpoints to make a spring, its called:
1. **Anchor**: Where the end point of the joint connects to _this_ object.
2. **Connected Anchor**: Where the end point of the joint connects to the _other_ object. 

You also need to configure:
1. **Distance**: the spring will try its best to keep the length of the spring (distance between Anchor and Connected Anchor) to be that value that you set. 
2. **Damping Ratio and Frequency**: to set the behaviour of the spring (hopefully now you find the stuffs taught in your Freshmore year useful). 

To create a nice bouncy spring,
1. **Set** the TopCollider's Rigidbody2D as the Connected Rigid Body property of the spring. 
2. **Tick** Auto Configure Connected Anchor and Auto Configure Distance properties. 
3. Set `Damping Ratio` to 1, and `Frequency` to 3 
4. Ensure that EdgeDetector's RigidBody2D **mass** is very small at 0.0001 as we set above, because we don't want so much force to nudge it, or allow it to *sink* when the game starts. 

You should have these settings at the end:

<img src="https://www.dropbox.com/s/uzzedzzgjkrzjyi/16.png?raw=1"  class="center_ninety"/>

## Layer and Collision Matrix for Physics2D 
**Before we can test**, we need to first ensure that the BoxCollider2D in TopCollider GameObject **does NOT** collide with the EdgeCollider2D in EdgeDetector GameObject.  
* The former is used to *collide with Mario* when he climbs on top of this box so he can *stand* on the box,
* while the latter is used to *bounce the box* (excite the spring) when he hits it from below. 

In order do have fine-grained collision tuning, we need to define the `Layer` of each object and set the engine's **Physics2D Collision Matrix**. On the top right hand corner of any GameObject inspector, notice there's a property called `Layer`. Just like a Tag, you can create your own Layer. It will be used by the Physics engine to determine **who can collide with each other.** 
* Create a new layer called `QuestionBox` and assign it to the `EdgeDetector` GameObject
* Assign the `Obstacles` layer you have created earlier to the `TopCollider` GameObject

Then, go to **Edit >> Project Settings >> Physics2D**. You should see some kind of Collision Matrix depending on how many different Layers you have set in your project. Right now you must only have the basics + `Obstacles` that you have created above. 

Untick collision between QuestionBox and Obstacles to disable collisions between the two.

<img src="https://www.dropbox.com/s/j0yq5gq038jh41m/15.png?raw=1"  class="center_ninety"/>

Now you can test your spring. While in Play mode, you can dynamically change the properties of the SpringJoint2D. Spend some time to play around with it to see how it works. You can also go to your Scene tab and dynamically drag around the TopCollider. Experiment around by unticking the auto configure distance and auto configure anchor properties to get a hang on how the spring works:

<img src="https://www.dropbox.com/s/82kvfabqsmg0vqz/spring.gif?raw=1"  class="center_ninety"/>

**Once you're satisfied, save the HittableSimple object as prefab.** 

## Physics Material
To get a more **bouncy** impact, you can create `Physics Material` and apply it on the Rigidbody of the EdgeDetector. It dictates the amount of **friction** or **bouncing** effects of colliding objects.  

Go to the Project tab, and under Materials folder, right click >> Create >> Physics Material. Name it `Bouncy` and set the `Bounciness` property to some positive value, e.g: 0.3.

Afterwards, apply this Physics Material onto the Material property in Edge Detector's Rigidbody2D component:

<img src="https://www.dropbox.com/s/qxyb6ywyfnk8lm2/17.png?raw=1"  class="center_ninety"/>

Test run to see if the box now looks _bouncier_. 

# Spawning GameObjects at Runtime
Now what's left is to spawn something at the top of the HittableSimple object when Mario hits its bottom edge. We can begin by creating the prefab of the object that we are going to spawn. 

Create a new 2D GameObject >> Sprites >> Square, name it `ConsumableMushroomSimple`:
* Change the Sprite property into any mushroom, e.g: `items_0`
* Add Rigidbody2D component (`Dynamic` type, with `Gravity Scale` of 10) 
* Add BoxCollider2D component and edit the collider to match the sprite
* Create a new `Layer` called `Spawned` and assign it to the mushroom
* Change its scale to be of the right size, then drag it to the Prefab folder 

You should have something like this in the end:
<img src="https://www.dropbox.com/s/i1es9tpw03m6x4o/18.png?raw=1"  class="center_ninety"/>

Now **delete** it from the Scene since we will only spawn it from the script and not in the beginning of the game. 

## Instantiate Prefab at Runtime
Double click on the `HittableSimple` **Prefab** (not the one on your scene!), and add a new C# Script component to its `EdgeDetector` GameObject, name the script: `QuestionBoxController.cs`. 

The logic for this script is:
1.  If the EdgeDetector has collided with Mario, then it has to *bounce* 
2.  And spawn the `ConsumableMushroomSimple` instantly, exactly **once** 
3. *When the box has finished bouncing*, it has to change Sprite (so the player wont hit it again)
4. And we have to disable the Spring as well (the box turns into a stationary object, just like the regular Brick) 

It is easy to do (1) and (2) above, as you might've guessed: simply write something inside `OnCollisionEnter2D` callback. Doing (3) requires us to *check* if the QuestionBox has turned stationary *after* briefly bouncing from being hit by Mario, and this is **not that trivial**, depending on how you choose to solve the problem. We will explain why later. 

Declare the following variables in `QuestionBoxController.cs`:
```java
public  Rigidbody2D rigidBody;
public  SpringJoint2D springJoint;
public  GameObject consumablePrefab; // the spawned mushroom prefab
public  SpriteRenderer spriteRenderer;
public  Sprite usedQuestionBox; // the sprite that indicates empty box instead of a question mark
private bool hit =  false;
```

Then implement `OnCollisionEnter2D`  callback function:
```java
void  OnCollisionEnter2D(Collision2D col)
{
	if (col.gameObject.CompareTag("Player") &&  !hit){
		hit  =  true;
		// spawn the mushroom prefab slightly above the box
		Instantiate(consumablePrefab, new  Vector3(this.transform.position.x, this.transform.position.y  +  1.0f, this.transform.position.z), Quaternion.identity);
	}
}
```

This will allow you to spawn the Mushroom right above the box when it's hit by Mario.  
* Attach this script to `EdgeDetector` GameObject under `HittableSimple` prefab (again, in prefab mode as shown in screenshot below! Not your game scene), 
* Link up all the required public variables with all the assets. 
* All assets except `Consumable Prefab` must be those in the Prefab's Tab and not the Assets tab.  
* You can find the mushroom prefab to link here under the Assets tab. 

<img src="https://www.dropbox.com/s/r5a7nshbtl94m47/19a.png?raw=1"  class="center_ninety"/>

## Consumable Mushroom
Now create a simple script yourself for the mushroom to dictate its behaviour as follows once spawned by the box:
* It will randomly move to the left or to the right at a **constant speed** 
* It **will not topple** (Z-rotation is constrained) 
	> Set its Rigidbody constraint properly
* However it will **change direction** when colliding with other objects (e.g: the Pipe. Quickly create some pipe prefabs of your own. You only need BoxCollider2D and a few sprites -- the pipe body and the pipe top -- for that) 
* It will **stop moving** when it collides with Mario (Player) 
	> Use `CompareTag()` function for the two points above
* It will collide normally with `Obstacles` (the bricks) or `Ground`, so it will travel above the bricks or ground normally
* It **will not collide** with the `TopCollider` GameObject (child of `HittableSimple`) so as **not** to affect the spring motion of the box
	> Ensure the mushroom's Layer (whatever you set it to be), do not collide with the `TopCollider`'s Layer
* It **springs** out of the box once instantiated (upwards)

The gif below summarises the required behavior of the spawned mushroom:

<img src="https://www.dropbox.com/s/4h6pg2ek3v94box/demomushroom.gif?raw=1"  class="center_ninety"/>

You probably can do all the above except the **first**: move the mushroom at constant speed (either to the left or to the right) and the last point (to give initial impulse force upwards). 

## Rigidbody2D MovePosition

To do the first point, we need to compute the supposed position of the mushroom in the next frame in the mushroom's script (whatever you name the script to be). Given `float speed`, current `Vector2 currentPosition` of the Mushroom, and `Vector2 currentDirection`  (`{1,0}` or `{-1,0}`, indicating movement towards the right or the left), we can compute the mushroom's next position as:

```java
Vector2 nextPosition = currentPosition + speed * currentDirection.normalized * Time.fixedDeltaTime;
``` 
where `Time.fixedDeltaTime` is simply the **interval in seconds** at which Physics frame rate updates. 

Afterwards, we can set the mushroom's Rigidbody2D position directly to be this `nextPosition` under `Update()` (instead of `FixedUpdate()` so as not to interfere with the Physics engine's Gravity computation). 

 ```java
Vector2 nextPosition = currentPosition + speed * currentDirection.normalized * Time.fixedDeltaTime;
rigidBody.MovePosition(nextPosition);
``` 

It is a bit *unclean* to do the above and mess with rigidBody's position **instead of giving complete control to Physics engine** (since it's a `dynamic` Rigidbody2D), so perhaps you can think of a **better** way to handle this, i.e: give the Mushroom an initial impulse Force to the left or to the right (zero linear friction set), and then flip the x-value of the velocity when it hits the other obstacles like the Pipe. 


> Note: although not written, it is explicit for you to **declare** `rigidBody` as `this` mushroom's rigidBody and instantiate it at the `Start` method using `GetComponent<Rigidbody2D>()` as usual.

## Rigidbody2D AddForce
Finally, it will be nice to add an **impulse** force upwards for the mushroom so it looks like it springs out of the box. Under the mushroom script's `Start()` method, you can add the instruction:

```java
rigidBody.AddForce(Vector2.up  *  20, ForceMode2D.Impulse);
```
where `20` is just a value which you can change as you like depending on the upwards force effect that you want. 

You might wonder what is the meaning behind `ForceMode2D.Impulse` and why didn't we use this mode instead:
```java
rigidBody.AddForce(Vector2.up  *  20, ForceMode2D.Force);
```

If you use `ForceMode2D.Force`, you might observe straight away that the effect is *not immediate* because the net amount of upwards force `Vector2.up  *  20` we wrote to be applied on the mushroom is automatically set as the amount of **TOTAL force to be applied over ONE second** (50 **physics** frame).
> By default on desktop, **Unity** runs the **FixedUpdate** at 50 times per second and the **Update** at 60 times per second. 

However it **DOES NOT** mean that the physics engine continuously apply this force over 1 whole second. It will apply only over the exact frames when you call it, e.g: over **0.02 seconds** if you only call it for a single frame.
> Therefore, if you were to call `rigidBody.AddForce(Vector2.up  *  20, ForceMode2D.Force)` continuously for **FIFTY** times (over 1 second), then the **total amount of force applied** on the mushroom body would've been the same as calling `rigidBody.AddForce(Vector2.up  *  20, ForceMode2D.Impulse)` for **ONE** frame (one time). 

In other words, `ForceMode2D.Impulse` tells the Unity Physics engine that you want to apply this much force **NOW**, where `ForceMode2D.Force` means that you want to apply this much force in total IF we were to call the `addForce()` method continuously, for exactly 50 times over 1 second. 

