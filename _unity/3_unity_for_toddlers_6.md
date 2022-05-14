---
title: Particle System 2D
permalink: /unity/unity_for_toddlers_6
key: unity-unity_for_toddlers_6
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

The final thing to learn about VFX in this lab is the **particle system**. Using Particle System component, you can simulate moving liquids, smoke, clouds, flames, magic spells, and a whole slew of other effects effectively. You can view their official tutorial [here](https://learn.unity.com/tutorial/introduction-to-particle-systems#:~:text=The%20Particle%20System%20in%20Unity,whole%20slew%20of%20other%20effects.), and in this Lab we are bringing you up to speed with two sample particles: landing jump "dust" and fire. Here's a demo (the fire glow is enhanced by the *bloom filter*):

<img src="https://www.dropbox.com/s/k8kxge0lhqvhfog/particle.gif?raw=1"  class="center_ninety"/>

## Creating Dust Particle
Create a new gameObject and attach a ParticleSystem component to it. Editing particle systems can be mainly done via the Inspector, by adjusting its properties accordingly. We need to first think how we want our particles to behave, for example, here's a few questions we can ask ourselves: How many particles can be emitted at a time? Do they have the same speed? Are they affected by gravity?

For this dust cloud particle, we definitely only want a few particles to be present at a time, and for it to fade over time quickly. They also must be instantiated only once -- in **bursts**. The setting of the particle system component must be pretty much as such:

<img src="https://www.dropbox.com/s/7tmgrjkdz5dw2vg/32.png?raw=1"  class="center_ninety"/>

* **Start Lifetime** is short, set at 0.5. We don't need the dust to linger for too long. 
* We also **disable** Play on Awake. We will trigger the particle at runtime. 
* **Looping** is also disabled. 
* The particle has a certain initial **speed** upwards (depending on the direction of "Shape") 
* Emission is done in **bursts**, in counts of 5 every 0.01 second (but since looping is disabled, it's going to happen just once when called to `Play()` via script later on).
* **Shape** is "Cone", as the dust gets bigger when it travel upwards. 
* **Color over lifetime** turns transparent, so we can no longer see the dust after a short period of time. 
* **Rotation** over lifetime is enabled, so the dust will rotate as it moves upwards and appear more natural. 
* Finally, we use a **custom** **material** (under Renderer).
	* Create a new material called Dust Cloud Material, with Shader set as: Universal Render Pipeline >> Particles >> Unlit
	* And load the Cloud sprite under Base Map
	* Set other properties as such:

<img src="https://www.dropbox.com/s/4uzvswrnpqtqcdx/33.png?raw=1"  class="center_ninety"/>

Place the DustCloud gameObject as a Child object under Mario. Now edit PlayerController.cs to trigger the DustCloud each time Mario lands on the ground using `dustCloud.Play();` where `dustCloud` is a reference to the Particle System component. Pretty sure you can do this by yourself easily. 


## Creating Fire Particle
The Fire particle is created via **Textured Sheet Animation**. A particle’s graphic need not be a still image. This module lets you treat the Texture as a **grid of separate sub-images** that can be played back as **frames** of animation. You can use your own sprites for this, or use the provided Cartoonfiretexture. If you're importing your own textures, make sure you **slice** them properly and set the texture's Sprite Mode property into **Multiple**.

Create a new Material called FireParticleMaterial using the Shader UniversalRenderingPipeline >> Particle >> Lit to make particles appear photorealistic. Set the Materials to have the following properties:

<img src="https://www.dropbox.com/s/jgjlfof76ovnbdr/34.png?raw=1"  class="center_ninety"/>

Now create a gameObject called Fire with a ParticleSystem component attached. Set its property as such:

<img src="https://www.dropbox.com/s/cq1nfhrnjl4l8nl/35a.png?raw=1"  class="center_ninety"/>

The main learning point here is to set the Texture Sheet Animation property as Grid, and to take up the Whole Sheet (the "texture" refers to the texture provided by the Material). You should observe the flame animating nicely by now. If you want Mario to be "burned" by it, you need to add a Collider component and implement a **Trigger** **callback** as we have learned before. 

# "Breakable" Prefabs
Another common feature that we can have in a game is to have "breakable" things, as shown:

<img src="https://www.dropbox.com/s/9crgqw9gfc167iu/break.gif?raw=1"  class="center_ninety"/>

There's a lot of different tips and tricks to do this. You're free to implement such effect however way you want: using particle system, using animation, or programmatically via the script. In this example, we will go back to the basics and use the latter. 

To create this simple 'breakable brick', we need **three** gameObjects:
1. A gameObject that contains the **Sprite Renderer** of the breakable brick and a regular **BoxCollider2D**.
2.  A **child** gameObject of (1) that contains a single **EdgeCollider2D** to detect collision only at the **bottom** of the brick. Tick its `IsTrigger` property.
3. A single "debris" **prefab**, which is simply a gameObject with RigidBody2D component attached (so gravity can act on it) and a Sprite Renderer. It has a single script `Debris.cs` which tells the Physics engine to propel this single debris outwards at the point of instantiation, and render itself invisible after some time. 

Now create a new script called `BreakBrick.cs,` to be attached in gameObject (1). Open the script and create a few variables:
1. A **boolean** to indicate whether this brick has been ***broken*** or not, 
2. A reference to the "debris" **prefab**

In `BreakBrick.cs,` implement `OnTriggerEnter2D` callback such that when the gameObject with this script collides with "Player", it spawns a few Debris prefab and disable all colliders: both edge collider and box collider, as well as the sprite renderer of the breakable brick:

```java
void  OnTriggerEnter2D(Collider2D col){
	if (col.gameObject.CompareTag("Player") &&  !broken){
		broken  =  true;
		// assume we have 5 debris per box
		for (int x =  0; x<5; x++){
			Instantiate(prefab, transform.position, Quaternion.identity);
		}
		gameObject.transform.parent.GetComponent<SpriteRenderer>().enabled  =  false;
		gameObject.transform.parent.GetComponent<BoxCollider2D>().enabled  =  false;
		GetComponent<EdgeCollider2D>().enabled  =  false;
	}
}
```

You are free to **destroy** this gameObject (1) afterwards.

The Script `Debris.cs` attached to a single debris prefab governs how each debris should behave. From the sample .gif above, each debris spawned will have an initial upward force with a random x-component, initial torque, and then it will gradually reduce in scale while being affected by Gravity. 

We need a **Coroutine** for this, because we need to slowly reduce the scale of the object **after each frame**, giving control back to Unity to perform other computations in the meantime. A sample implementation is as such:

```java
private  Rigidbody2D rigidBody;
private  Vector3 scaler;

// Start is called before the first frame update
void  Start()
{
	// we want the object to have a scale of 0 (disappear) after 30 frames. 
	scaler  =  transform.localScale  / (float) 30 ;
	rigidBody  =  GetComponent<Rigidbody2D>();
	StartCoroutine("ScaleOut");
}

IEnumerator  ScaleOut(){

	Vector2 direction =  new  Vector2(Random.Range(-1.0f, 1.0f), 1);
	rigidBody.AddForce(direction.normalized  *  10, ForceMode2D.Impulse);
	rigidBody.AddTorque(10, ForceMode2D.Impulse);
	// wait for next frame
	yield  return  null;

	// render for 0.5 second
	for (int step =  0; step  < 30; step++)
	{
		this.transform.localScale  =  this.transform.localScale  -  scaler;
		// wait for next frame
		yield  return  null;
	}

	Destroy(gameObject);

}
```

You can enhance the effect by adding **sound effect** at the moment of impact, or allow the debris to bounce off other things in the scene. 
