---
title: Coroutines
permalink: /unity/unity_for_babies_3
key: unity-unity_for_babies_3
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


Up until this point, we need to **disable** the `HittableSimple`'s spring and Rigidbody2D ***after*** it has finished bouncing. The problem with this is that we do not know *when* exactly the box has finished bouncing. We can continuously check under its script's `Update()` method after `hit == true`, or we can use an alternative called **Coroutines**. 
> A coroutine is like a function that has the ability to **pause** execution and return control to Unity but then to **continue where it left off on the following frame**. A normal function like `Update()` cannot do this and must run into completion before returning control to Unity. 

You can declare a Coroutine like this:
```java
IEnumerator  functionName(){
	//implementation here
}
```
and call it like this: `StartCoroutine(functionName());`.

Paste the following Coroutine and regular function inside `QuestionBoxController.cs`:
```java
bool  ObjectMovedAndStopped(){
	return  Mathf.Abs(rigidBody.velocity.magnitude)<0.01;
}

IEnumerator  DisableHittable(){
	if (!ObjectMovedAndStopped()){
		yield  return  new  WaitUntil(() =>  ObjectMovedAndStopped());
	}

	//continues here when the ObjectMovedAndStopped() returns true
	spriteRenderer.sprite  =  usedQuestionBox; // change sprite to be "used-box" sprite
	rigidBody.bodyType  =  RigidbodyType2D.Static; // make the box unaffected by Physics

	//reset box position
	this.transform.localPosition  =  Vector3.zero;
	springJoint.enabled  =  false; // disable spring
}
```
The instruction `yield return new <something>` returns control to Unity until that `<something>` condition happens. We can wait for a few seconds:  `yield return new WaitForSeconds(0.1f)`, or [wait until end of frame](https://docs.unity3d.com/ScriptReference/WaitForEndOfFrame.html), etc. It will continue with the **next** instruction when resumed, which is `spriteRenderer.sprite  =  usedQuestionBox;` for the above example. 

Another common way to `yield` is `yield return null`.
> **`yield return null`** waits for the next frame and continue execution from this line
 
For example, consider this sample code that gradually fades the color of an object,
```java
IEnumerator Fade() { 
	for (float ft = 1f; ft >= 0; ft -= 0.1f) 
	{ 
		Color c = renderer.material.color; 
		c.a = ft; 
		renderer.material.color = c; 
		yield return null; 
	} 
}
```
Without `yield return null`, Unity runs the loop **fully for the amount of time in one frame**, therefore blocking the editor and the intermediate values will never be seen. The object will disappear instantly.

> Hence, in order to see the fading effect, the effect of each loop must be seen per frame -- rendered out to the viewer. Using `yield return null` returns the control back to Unity, and the instruction (`for` loop check) will be executed back in the next frame.  

Now back to our game, we can call the Coroutine inside `OnCollisionEnter2D`, right after we instantiate the mushroom inside `QuestionBoxController.cs`:
```java
void  OnCollisionEnter2D(Collision2D col)
{
	// Debug.Log("OnCollisionEnter2D");
	if (col.gameObject.CompareTag("Player") &&  !hit){
		hit  =  true;
		Instantiate(consumablePrefab, new  Vector3(this.transform.position.x, this.transform.position.y  +  1.0f, this.transform.position.z), Quaternion.identity);
		StartCoroutine(DisableHittable());
	}
}
```

> Note that Coroutine is **NOT** creation of new threads. They are executed **sequentially** until they  `yield`. The engine will check all yielded coroutines as part of its own main loop (at what point exactly depends on the type of  `yield`), continue them one after another until their next  `yield`, and then proceed with the main loop. Therefore Coroutine introduces **concurrency** and not parallelism. 

## Has the Question Box moved?
One problem with the above implementation is that we cannot know for sure if the box has bounced up (has moved) *sufficiently high* before `ObjectMovedAndStopped()` returns **true**, depending on *where* Mario hits the box. It can be the case that Mario so very slightly collides with the question box's edge but didn't give sufficient energy to bounce the spring attached to the hittable box. 

To fix this quickly, we can add **more** upwards force when collision is detected, and be sure that the `HittableSimple` box bounces at least of a certain amount regardless of Mario's momentum. The complete `OnCollisionEnter2D` implementation is as follows:

```java
void  OnCollisionEnter2D(Collision2D col)
{
	if (col.gameObject.CompareTag("Player") &&  !hit){
		hit  =  true;
		// ensure that we move this object sufficiently 
		rigidBody.AddForce(new  Vector2(0, rigidBody.mass*20), ForceMode2D.Impulse);
		// spawn mushroom
		Instantiate(consumablePrefab, new  Vector3(this.transform.position.x, this.transform.position.y  +  1.0f, this.transform.position.z), Quaternion.identity);
		// begin check to disable object's spring and rigidbody
		StartCoroutine(DisableHittable());
	}
}
```

If everything's implemented correctly, you should have this nice bouncy box effect that's no longer moving or spawn anything after it was hit once:

<img src="https://www.dropbox.com/s/ulvtr8wjc6sbusi/bouncefinal.gif?raw=1"  class="center_ninety"/>

> **Disclaimer**: Note that this is not the only way to implement the entire bouncing effect of the box. There's a lot of other alternatives out there, which you are free to choose and implement. These specific methods of using SpringJoint2D, Rigidbody2D methods (addForce, etc), and Coroutines are selected because we want to teach the Unity newbies these concepts by incorporating it into our sample game. 



# Moving the Camera

The second thing to do to complete the game for this tutorial is to let the Camera follow the player, but clamped so that it doesn't go out of screen too much the left or too much the right. Create a new script called `CameraController.cs` and declare the following variables:

```java
public  Transform player; // Mario's Transform
public  Transform endLimit; // GameObject that indicates end of map
private  float offset; // initial x-offset between camera and Mario
private  float startX; // smallest x-coordinate of the Camera
private  float endX; // largest x-coordinate of the camera
private  float viewportHalfWidth;
```

In the `Start()` method, we instantiate a few things, but the highlight lies on the part where we need to get the **world coordinate** of the bottom-left point of the Camera's **viewport** using `ViewportToWorldPoint`. The reason we do this is because we need to find out what exactly is the world coordinate (x-coordinate specifically) of the leftmost point of the viewport. 
> We use it later on to prevent the camera to move *too much to the left.* 

```java
void  Start()
{
	// get coordinate of the bottomleft of the viewport
	// z doesn't matter since the camera is orthographic
	Vector3 bottomLeft =  Camera.main.ViewportToWorldPoint(new  Vector3(0, 0, 0));
	viewportHalfWidth  =  Mathf.Abs(bottomLeft.x  -  this.transform.position.x);

	offset  =  this.transform.position.x  -  player.position.x;
	startX  =  this.transform.position.x;
	endX  =  endLimit.transform.position.x  -  viewportHalfWidth;
}
```

Then under the `Update()` method, the camera constantly follow the player unless it has reached the ends of the game map:

```java
void  Update()
{
	float desiredX =  player.position.x  +  offset;
	// check if desiredX is within startX and endX
	if (desiredX  >  startX  &&  desiredX  <  endX)
	this.transform.position  =  new  Vector3(desiredX, this.transform.position.y, this.transform.position.z);
}
```

Create an empty GameObject called `EndLimit` with an EdgeCollider2D to prevent Mario from going  over too much to the right:

<img src="https://www.dropbox.com/s/jc0qffh8jq8b64a/20.png?raw=1"  class="center_ninety"/>

> You can do the same for the left side as well, right at the left side of the Camera's ViewPort. In the screenshot above, we name it `StartLimit`. 

Then in the Camera's inspector, link up the references of Mario's Transform and EndLimit's Transform:

<img src="https://www.dropbox.com/s/to83qz57x4o5yvc/21.png?raw=1"  class="center_ninety"/>

**Test run and you shall have the Camera gloriously following Mario around the Scene**. 

# Destroy GameObject

One final thing to do is to ensure that the `ConsumableMushroomSimple` Prefab is **Destroyed** once it **goes out of view.** It is very simple to do this as there's a callback dedicated for it: `OnBecameInvisible`. 

> From Unity's documentation: `OnBecameInvisible` is called when the **renderer** is no longer visible by any camera. This message is sent to all scripts attached to the renderer.  [OnBecameVisible](https://docs.unity3d.com/ScriptReference/MonoBehaviour.OnBecameVisible.html)  and  [OnBecameInvisible](https://docs.unity3d.com/ScriptReference/MonoBehaviour.OnBecameInvisible.html)  is useful to avoid computations that are only necessary when the object is visible.

Open the script controlling `ConsumableMushroomSimple` and add the following method, and we're done! The mushroom disappears once it's no longer visible by the camera. This callback obviously only works on gameObjects that have `Renderer` attached to it. 

```java
void  OnBecameInvisible(){
	Destroy(gameObject);	
}
```

> Some newbies will make the mistake `Destroy(this)` thinking that `this` is the `gameObject`. This is not true. `this` refers to `this (script) instance` instead. 


 