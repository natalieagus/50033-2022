---
title: Scriptable Object and Managers
permalink: /unity/unity_for_children_3
key: unity-unity_for_children_3
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

# Scriptable Objects

A ScriptableObject is a **data container** that you can use to **save** large amounts of data, independent of class instances. An example scenario where this will be useful is when your game needs to instantiate tons of Prefab with a Script component that stores unchanging variables. We can save memory by storing these data in a ScriptableObject instead and these Prefabs can refer to the content of the ScriptableObject at runtime. 

ScriptableObject is also useful to store standard values for your game, such as reset values, max health for each character, cost of items, etc. In the later weeks, we will also learn how to utilise ScriptableObjects to create a handy Finite State Machine. 

To begin creating this data container, create a new script and call it `GameConstants.cs`. Instead of inheriting `MonoBehavior` as usual, we let it inherit `ScriptableObject`:

```java
using UnityEngine;

[CreateAssetMenu(fileName =  "GameConstants", menuName =  "ScriptableObjects/GameConstants", order =  1)]
public  class GameConstants : ScriptableObject
{
	// set your data here
}
```

The header `CreateAssetMenu..` allows us to create instances of this class in the Project. Proceed by declaring a few constants that might be useful for your project inside the class, for example:

```java
// for Scoring system
int currentScore;
int currentPlayerHealth;

// for Reset values
Vector3 goombaSpawnPointStart = new Vector3(2.5f, -0.45f, 0); // hardcoded location
// .. other reset values 

// for Consume.cs
public  int consumeTimeStep =  10;
public  int consumeLargestScale =  4;
  
// for Break.cs
public  int breakTimeStep =  30;
public  int breakDebrisTorque =  10;
public  int breakDebrisForce =  10;
  
// for SpawnDebris.cs
public  int spawnNumberOfDebris =  10;
  
// for Rotator.cs
public  int rotatorRotateSpeed =  6;
  
// for testing
public  int testValue;
```

Now you can instantiate the object by right clicking on the Project window then >> Create, since we have declared `CreateAssetMenu`. 

<img src="https://www.dropbox.com/s/umvnm2ccxd0vkvv/10.png?raw=1"  class="center_ninety"/>

You can also edit the values of the variables in the inspector. These values persist, so you can store something in these data containers during runtime, such as the player's highest score. 

To use it in any script, simply declare it as a public variable and link it up in the inspector. Below is a simple sample script that can rotate the object it is attached to, at a rotational speed as set in `gameConstants`. 

```java
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public  class Rotator : MonoBehaviour
{
	public  GameConstants gameConstants;
	private  Vector3 rotator;

	// Start is called before the first frame update
	void  Start()
	{
		rotator  =  new  Vector3(0, gameConstants.rotatorRotateSpeed, 0);
	}

	// Update is called once per frame
	void  Update()
	{
		//Rotate
		this.transform.rotation  =  Quaternion.Euler(this.transform.eulerAngles  -  rotator);
	}
}
```

You can change these values conveniently to find the right "feel" for your game during testing.

# Creating Managers

There are many design patterns to create a game manager. You can even have a hierarchy of managers in the game, depending on the role of each manager: managing score, managing enemies, managing powerups, etc. However, sometimes we may cut corners because we simply do not have enough time to read about all of them. In this section, we will create a working game manager, and we a few C# tricks. By no means we claim that this is the best way to create a game manager, and it is simply a tool to introduce to you the concept of a basic *manager*, and a few fancy C# stuffs. 

> If you already know most of the concepts taught in this section, then the best way to proceed from here and learn further is to reverse engineer how others implement their managers. You can start with several advanced Unity tutorial projects freely available online. 

The idea is to make sure that we have a centralised way to manage score, enemies, players, and pretty much everything important in the Scene. Suppose we want a game with three basic system:
1. **Scoring system:** enemies spawned from the pool can be killed and this will increase the score. 
2. **Damage system:** player will die if it collides with the enemies from the side before managing to kill the enemy. 
3. **Powerup system:** player can collect powerups and use it to boost something for a fixed period of time. 

There has to be a script to manage all these game-wide logistics, which we can call the *managers*, while scripts attached to the prefabs or gameobjects simply manage the gameobject it is attached to. 

## Scoring System

Previously in our first lab, we manage the ScoreText UI directly from the PlayerController script. We need a better way to do this: if Player scores anything, it has to notify a manager to update the score value. 

Ensure that you have a UI Text score in your Scene, placed somewhere convenient in an *Screen Space - Overlay* render mode as such:

<img src="https://www.dropbox.com/s/ocosq33ddfxej1g/11.png?raw=1"  class="center_ninety"/> 

## GameManager.cs
Create a new script called `GameManager.cs`, declare and implement the following:

```java
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public  class GameManager : MonoBehaviour
{
	public  Text score;
	private  int playerScore =  0;
	
	public  void  increaseScore(){
		playerScore  +=  1;
		score.text  =  "SCORE: "  +  playerScore.ToString();
	}
}
```

Obviously the method `increaseScore` will have to be called whenever the player legally kill one of the enemies. We will have eventually a few other managers in the game, such as `SpawnManager` and `PowerupManager`, and we do not want any script to be able to call methods from any manager. Therefore we need some kind of `CentralManager` that helps to call other manager methods such as this `increaseScore` so that we have less management headache. 

## CentralManager.cs
Create a new script called `CentralManager.cs` and implement the following:

```java
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// this has methods callable by players
public  class CentralManager : MonoBehaviour
{
	public  GameObject gameManagerObject;
	private  GameManager gameManager;
	public  static  CentralManager centralManagerInstance;
	
	void  Awake(){
		centralManagerInstance  =  this;
	}
	// Start is called before the first frame update
	void  Start()
	{
		gameManager  =  gameManagerObject.GetComponent<GameManager>();
	}

	public  void  increaseScore(){
		gameManager.increaseScore();
	}
}
```

## C#: Static variable
The `CentralManager`has a reference to current GameManager instance in the scene, and this can be easily set in in the inspector. However,  since every other script has to refer to the current `CentralManager` instance in the Scene for help, then a `static` variable: `centralManagerInstance` is declared, which value is set to `this` upon instantiation of `CentralManager`. This way we do not have to link up the current `CentralManager` reference to every script in the Scene. 

`centralManagerInstance` is a static variable, any other script can **conveniently** refer to `this` instance of `CentralManager` using the class:

```java
CentralManager.centralManagerInstance.method(parameters...)
``` 

## Create EnemyController.cs Script for Enemy Prefab
Initially we have created `goombaEnemy` and `greenEnemy` prefabs. We now need a script to control their movements and behaviour. It is up to you how you want to implement this script, but for example let's just implement one similar to our first lab, that is to create enemies that patrol back and forth between Point A and B, where Point A and B are at at most 5 x-units away from its initial `transform.position.x`. 

Here's a sample quick implementation to move the enemy back and forth for a fixed distance. Set up the appropriate constants for `maxOffset` and `enemyPatrolTime` in `gameConstants` accordingly. 

```java
using System.Collections;
using UnityEngine;

public  class EnemyController : MonoBehaviour
{
	public  GameConstants gameConstants;
	private  int moveRight;
	private  float originalX;
	private  Vector2 velocity;
	private  Rigidbody2D enemyBody;
	
	void  Start()
	{
		enemyBody  =  GetComponent<Rigidbody2D>();
		
		// get the starting position
		originalX  =  transform.position.x;
	
		// randomise initial direction
		moveRight  =  Random.Range(0, 2) ==  0  ?  -1  :  1;
		
		// compute initial velocity
		ComputeVelocity();
	}
	
	void  ComputeVelocity()
	{
			velocity  =  new  Vector2((moveRight) *  gameConstants.maxOffset  /  gameConstants.enemyPatroltime, 0);
	}
  
	void  MoveEnemy()
	{
		enemyBody.MovePosition(enemyBody.position  +  velocity  *  Time.fixedDeltaTime);
	}

	void  Update()
	{
		if (Mathf.Abs(enemyBody.position.x  -  originalX) <  gameConstants.maxOffset)
		{// move goomba
			MoveEnemy();
		}
		else
		{
			// change direction
			moveRight  *=  -1;
			ComputeVelocity();
			MoveEnemy();
		}
	}
```

Now we need to check if it collides with the Player. Ensure that the BoxCollider2D of the enemy prefabs has `isTrigger` property **checked**. Then implement `OnTriggerEnter2D` method:

```java
	void  OnTriggerEnter2D(Collider2D other){
		// check if it collides with Mario
		if (other.gameObject.tag  ==  "Player"){
			// check if collides on top
			float yoffset = (other.transform.position.y  -  this.transform.position.y);
			if (yoffset  >  0.75f){
				KillSelf();
			}
			else{
				// hurt player, implement later
			}
		}
	}
```

The idea is to check if the player's `y` location is higher than the enemy's (that's how we know the Player is *stomping* the enemy from above). If yes, then we the enemy is killed:
```java
	void  KillSelf(){
		// enemy dies
		CentralManager.centralManagerInstance.increaseScore();
		StartCoroutine(flatten());
		Debug.Log("Kill sequence ends");
	}
```

A little Coroutine called `flatten` gradually animate the enemy being flattened onto the ground as follows:
```java
	IEnumerator  flatten(){
		Debug.Log("Flatten starts");
		int steps =  5;
		float stepper =  1.0f/(float) steps;

		for (int i =  0; i  <  steps; i  ++){
			this.transform.localScale  =  new  Vector3(this.transform.localScale.x, this.transform.localScale.y  -  stepper, this.transform.localScale.z);

			// make sure enemy is still above ground
			this.transform.position  =  new  Vector3(this.transform.position.x, gameConstants.groundSurface  +  GetComponent<SpriteRenderer>().bounds.extents.y, this.transform.position.z);
			yield  return  null;
		}
		Debug.Log("Flatten ends");
		this.gameObject.SetActive(false);
		Debug.Log("Enemy returned to pool");
		yield  break;
	}
```

The value `gameConstants.groundSurface` is the global y-position of a point directly above the ground. In our case, the groundSurface is right at `y = -1`. 

> Note that Coroutines are not any form of multi Threading and it does not allow any kind of Parallel execution. 

The debug messages are printed in this order: 
	> * `Flatten starts`, 
	> * `KillSequenceEnds`, 
	> * `Flatten ends`, then 
	> * `Enemy returned to pool`, 

This which shows that the method `KillSelf` **does NOT have to wait until Coroutine flatten ends** to resume execution, thus allowing **asynchronous** execution.

However, `onTriggerEnter2D` can only be called again once  `KillSelf` exits because `KillSelf` is synchronous with `OnTriggerEnter2D`. 

At each `yield` in `flatten`, control is **returned** to Unity and it will still call **other functions** in `Enemy Controller` script such as `Update` and `FixedUpdate`. It is important to **never** call too many Coroutines during `Update` or `FixedUpdate` as it may cause performance issues. 

To test run, create a few gameObjects a  parent gameObject named `Managers` with two children (and no other components): 
* GameManager with `GameManager.cs` attached
* CentralManager with `CentralManager.cs` attached 

Set up the inspectors appropriately: link up `Score` Text in. `GameManager.cs` and `GameManager` gameobject in `CentralManager.cs`. Do not forget to link up `gameConstants` whenever you refer to it in the enemy prefabs or everywhere else in your code when you refer to it. 

Test run and you should observe the enemies getting flattened whenever the Player lands right on top if it, and the score is increased. 

## Damaging Player
Create another method in `CentralManager.cs` which will be called whenever the Player collides with the enemies sideways:
```java
public  void  damagePlayer(){
	gameManager.damagePlayer();
}
```

Now implement the same method in `GameManager.cs`:
```java
public  void  damagePlayer(){
	OnPlayerDeath();
}
```

## C#: Delegates and Events
What is `OnPlayerDeath`? It is not a method implemented in `GameManager.cs` but it is an **event**, of which all of its **subscribers** will cast the subscribed method whenever `GameManager` calls `OnPlayerDeath()`. 

Event is a special type of **delegate**. A delegate is a **reference** **pointer** to a method. It allows us to treat method as a variable and pass method as a variable for a callback. When a delegate gets called, it notifies all methods that **reference** the delegate. 

> The basic idea behind them is exactly the same as a subscription magazine. Anyone can subscribe to the service and they will receive the update at the right time automatically.

You can declare a delegate with the `delegate` keyword and specifies its signature (return type and parameters):
```java
public  delegate  returnType name(parameter1, parameter2, ...);
```

Declare the following delegate in  `GameManager`.cs:
```java
public  delegate  void gameEvent();
```

To allow other scripts to subscribe to this delegate, we need to create an instance of that delegate, using the `event` keyword:
```java
public  static  event  gameEvent OnPlayerDeath;
```

> Note that we can also use the delegate directly using its name: `public static gameEvent OnPlayerDeath`, but **without** the keyword `event` then `OnPlayerDeath()` can be **cast** by anyone (unless it is not `public`, but that will mean that not every other script can subscribe to it). If we want only the *owner* of the delegate to cast, such as this `GameManager`, then the `event` keyword is used. 

We instantiate two events from `gameEvent` delegate. Any other script can now subscribe to this event. Open `EnemyController.cs` and implement the following under the `Start()` method. 

> Remember that the event `OnPlayerDeath`  is set as **static**, so we can conveniently refer to them using the classname `GameManager` instead of finding the instance reference during runtime.  

```java 
// subscribe to player event
GameManager.OnPlayerDeath  +=  EnemyRejoice;
```

`EnemyRejoice` is simply a method in in `EnemyController.cs` implemented as:
```java
// animation when player is dead
void  EnemyRejoice(){
	Debug.Log("Enemy killed Mario");
	// do whatever you want here, animate etc
	// ...
}
```

`EnemyRejoice` has the **same signature** as `delegate gameEvent` in `GameManager`, so it can subscribe to `OnPlayerDeath` event derived from `gameEvent` delegate. 

You can have multiple methods subscribed to the same event (that's the point of subscription!), so open `PlayerController.cs` and implement a method that you want to be called when Mario dies:
```java
void  PlayerDiesSequence(){
	// Mario dies
	Debug.Log("Mario dies");
	// do whatever you want here, animate etc
	// ...
}
```

Then subscribe to the event at `Start()` in `PlayerController.cs`:

```java
GameManager.OnPlayerDeath  +=  PlayerDiesSequence;
```

Finally, when `damagePlayer()` is called in `GameManager.cs`, this will cast the **delegate** `OnPlayerDeath()`, which in turn will call `EnemyRejoice()` and `PlayerDiesSequence()` **both** as they're both subscribed to the event. The order of execution depends on the order of subscription, and they're **sequential**.

You can learn more about delegate and events from Unity tutorials [here](https://learn.unity.com/tutorial/events-uh). 

Now propagate these chain of function calls at the `EnemyController.cs` by implementing the **else** clause in the `OnTriggerEnter2D` function we did earlier: 

```java
void  OnTriggerEnter2D(Collider2D other){
	// check if it collides with Mario
	if (other.gameObject.tag  ==  "Player"){
		// check if collides on top
		float yoffset = (other.transform.position.y  -  this.transform.position.y);
		if (yoffset  >  0.75f){
			KillSelf();
		}
		else{
			// hurt player
			CentralManager.centralManagerInstance.damagePlayer();
		}
	}
}
```

Test run and you should at least see the Debug messages printed out whenever Player collides with the enemy in any way except from stomping it from above. 

### Checkoff information
Create a new event that's to be casted when `increaseScore()` in GameManager is called, such that it results in spawning of one new enemy. The class that should subscribe to this new event is `SpawnManager.cs` we created earlier. 

## PowerupManager.cs
Create a new script called `PowerupManager.cs` which will be used to manage the powerups that Mario can *collect* and *consume* anytime at will. 

Before that, open the Consumable Mushroom prefab that you have created in the previous lab and another copy. Rename them both into `RedMushroom` and `OrangeMushroom` and select two different sprites (you can name it however you want actually, it does not matter. We will only use these names for easy referencing in this tutorial). 

You should already have a script that controls its direction of patrol as per previous lab.  You might now want to add a **collision** check with Player, and a state `bool collected` to indicate that the powerup is **collected** so that **you don't need to move the mushroom anymore when collected**. Also add some kind of visual feedback upon collision. Here's a simple example: the mushroom will slightly enlarge and then scaled in. 

<img src="https://www.dropbox.com/s/cwvjyz38p6nkjrb/consume.gif?raw=1"  class="center_ninety"/>

## C#: Interface
It is very common in a game to have various types of powerups, but they should have common methods that will be called by other scripts such as `cast` or `consume`, etc. To do this more uniformly, we can utilise an `interface`. Interface members must be **public** by default, because they're meant to define the public API of a type (hence the name *interface*: a **contract** meant to be use by other classes). 

Create a new script called `ConsumableInterface.cs`, where we can declare method signatures:

```java
using UnityEngine;

public  interface ConsumableInterface{
	void  consumedBy(GameObject player);
}
```

Now create two more scripts, `RedMushroom.cs` and `OrangeMushroom.cs` that will implement this interface:

```java
using System.Collections;
using UnityEngine;

public  class RedMushroom : MonoBehaviour, ConsumableInterface
{
	public  Texture t;
	public  void  consumedBy(GameObject player){
		// give player jump boost
		player.GetComponent<PlayerController>().upSpeed  +=  10;
		StartCoroutine(removeEffect(player));
	}

	IEnumerator  removeEffect(GameObject player){
		yield  return  new  WaitForSeconds(5.0f);
		player.GetComponent<PlayerController>().upSpeed  -=  10;
	}
}
```

As you can see, the RedMushroom, when consumed will give the player jump boost for 5 seconds. 

The OrangeMushroom will give the player a speed boost for 5 seconds:
```java
using System.Collections;
using UnityEngine;

public  class OrangeMushroom : MonoBehaviour, ConsumableInterface
{
	public  Texture t;
	public  void  consumedBy(GameObject player){
		// give player jump boost
		player.GetComponent<PlayerController>().maxSpeed  *=  2;
		StartCoroutine(removeEffect(player));
	}

	IEnumerator  removeEffect(GameObject player){
		yield  return  new  WaitForSeconds(5.0f);
		player.GetComponent<PlayerController>().maxSpeed  /=  2;
	}
}
```
Attach `RedMushroom.cs` and `OrangeMushroom.cs` to your respective consumable prefabs (the prefab that's spawned and patrolling around after hitting ? boxes) respectively. 

Set `Texture t` in the inspector to be a picture you want to indicate in the UI later on on the availability of this powerup. 

### PowerupUI
Now what is left is to indicate how many powerups are usable, and to bind "consume" of powerup with a key. Create a UI gameobject as follows:

<img src="https://www.dropbox.com/s/7dxpmqm0f8t4q3i/12.png?raw=1"  class="center_ninety"/>

The two children objects: PowerupSlot1 and PowerupSlot2 have RawImage as a component. We will load RedMushroom texture at Slot2 and OrangeMushroom texture at Slot1 whenever Mario collected them (but not yet consumed). 

Create a script called `PowerUpManager.cs`:
```java
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public  class PowerUpManager : MonoBehaviour
{
	public  List<GameObject> powerupIcons;
	private  List<ConsumableInterface> powerups;

	// Start is called before the first frame update
	void  Start()
	{
		powerups  =  new  List<ConsumableInterface>();
		for (int i =  0; i<powerupIcons.Count; i++){
			powerupIcons[i].SetActive(false);
			powerups.Add(null);
		}
	}
}
```

Here we store a list of existing powerups *interfaces*, and initialise them as *null* at first because Mario hasn't collected anything. We also disable the UI placeholder. 

Write two methods to add and remove powerups from the list:

```java
public  void  addPowerup(Texture texture, int index, ConsumableInterface i){
	Debug.Log("adding powerup");
	if (index  <  powerupIcons.Count){
		powerupIcons[index].GetComponent<RawImage>().texture  =  texture;
		powerupIcons[index].SetActive(true);
		powerups[index] =  i;
	}
}

public  void  removePowerup(int index){
	if (index  <  powerupIcons.Count){
	powerupIcons[index].SetActive(false);
	powerups[index] =  null;
	}
}
```

## C#: Switch statements
Finally, add these two methods in `PowerUpManager.cs` to consume the powerups when key Z or X is pressed. You can set your own key binding. This is just an example:

```java
void  cast(int i, GameObject p){
	if (powerups[i] !=  null){
		powerups[i].consumedBy(p); // interface method
		removePowerup(i);
	}
}

public  void  consumePowerup(KeyCode k, GameObject player){
	switch(k){
		case  KeyCode.Z:
			cast(0, player);
			break;
		case  KeyCode.X:
			cast(1, player);
			break;
		default:
			break;
	}
}
```

`consumedBy` is a method that's guaranteed to be implemented by any class having this `ConsumableInterface`. **It is a good way to implement a standardized method without any ambiguity on how to call it**, not to mention that it is neat to have such standardized interface for similar object types and abstract out the implementation of its effects within the method. It will be quite a nightmare if each powerup has its own method signature. 

Now obviously `consumePowerup` and `addPowerup`  should be called from outside `PowerUpManager.cs` script. Remember that we do not necessarily want any other script to call any Managers as they please, so we need to add these methods to `CentralManager.cs` and let other Scripts call them via `CentralManager`:

```java
// add reference to PowerupManager
public  GameObject powerupManagerObject;
private  PowerUpManager powerUpManager;

// instantiate in start
powerUpManager  =  powerupManagerObject.GetComponent<PowerUpManager>();

public  void  consumePowerup(KeyCode k, GameObject g){
	powerUpManager.consumePowerup(k,g);
}

public  void  addPowerup(Texture t, int i, ConsumableInterface c){
	powerUpManager.addPowerup(t, i, c);
}
```

The script that will call `addPowerup` is both RedMushroom.cs and OrangeMushroom.cs, attached to the consumable prefab and called when they collide with Mario:
```java
void  OnCollisionEnter2D(Collision2D col)
{
	if (col.gameObject.CompareTag("Player")){
		// update UI
		CentralManager.centralManagerInstance.addPowerup(t, index, this);
		GetComponent<Collider2D>().enabled  =  false;
	}
}
```
where `index` can be set to `0` or `1` depending on which key, `z` or `x` you choose to activate the powerups. 

The script that will call `userPowerup` is `PlayerController.cs`, under the `Update()` method:

```java
if (Input.GetKeyDown("z")){
	CentralManager.centralManagerInstance.consumePowerup(KeyCode.Z,this.gameObject);
}

if (Input.GetKeyDown("x")){
	CentralManager.centralManagerInstance.consumePowerup(KeyCode.X,this.gameObject);
}
```

As a summary: 
* When Mario collides with Red/Orange Mushroom, `OnCollision2D` callback in `Red/OrangeMushroom.cs` will call `addPowerup` in CentralManager, and pass the reference for Texture `t` and `this instance` eventually to `PowerupManager`. This will show the UI that a powerup is available. 
* When Mario presses `z` or `x`, it will call `consumePowerup` in CentralManager, and pass the reference of itself. This will eventually reach `PowerupManager`'s `consumePowerup` and it will call the `consumedBy` method implemented in both `RedMushroom` and `OrangeMushroom` if any of these powerups are available. 


