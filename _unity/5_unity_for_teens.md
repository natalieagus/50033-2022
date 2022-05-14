---
title: Unity for Teens
permalink: /unity/unity_for_teens
key: unity-unity_for_teens
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


**Learning Objectives:** Advanced Managements
- Persistence between Scenes using Singletons
- Game Architecture with Scriptable Objects
	- ScriptableObject Event System
	- UnityEvents, Custom UnityEvents stored in  ScriptableObjects
	- Custom EventListeners to attach to GameObjects
	- Sample ScriptableObject scripts for Inventory system
-   C#:
	-  Using properties,
	- Method overloading,
 
*Prepare your brain.* Today's objectives may look pretty simple but it is our most difficult lab yet. We will be learning an entire game architecture with Scriptable Objects. It will be painful, but worth it. It will make your game modular and easy to develop, debug, and manage. It wasn't taught earlier since you need to have sufficient fundamental knowledge to know what goes on in the Scene. This architecture requires immense planning and more-than-basic knowledge. You're now ready. 




# Introduction
Most things that will be done in this lab will not be immediately visible (unlike Shaders, spawning new enemies, etc that we have done in the previous weeks), but they can be handy tool to manage your project.

As usual, veterans can head straight to the [Checkoff](https://natalieagus.github.io/50033/2021/01/05/unityforteens.html#checkoff) section and implement the required features any way you deem fit.
> Quick check: you have learned about **ScriptableObject Event System** and **ScriptableObject Game Architecture**, you're good to skip the tutorial.

You still need to know the contents of this lab though, as that will be used for our quizzes.

# GameObject Persistence Between Scenes
When we switch from one game scene to another, we will **destroy** all GameObjects (along with its components) in the previous scene. Sometimes we want some GameObjects to persist, or more specifically: its data or state. We can definitely use Scriptable Objects to store some relevant data like GameScore, Player items, stats, current buffs, etc that can be carried over to the next scene, and then load it up when the new scene is ready. We can present the player with some kind of loading screen when we load these information for the new GameObject instances to process. 

An alternative to this will be to allow some GameObjects to be not destroyed upon loading of a new scene. We can create a script that implement the Singleton pattern so that the GameObject (along with its children) is not destroyed upon load. Note that this will only apply on **root** GameObject. 

**Create a new scene** and name it something that makes sense. In this example, we create a new scene and name it `MarioLevel2`. It is just a simple scene with a few platforms and question boxes from prefabs we created in the earlier tutorials. 

<img src="https://www.dropbox.com/s/sj3butbjgl8dewt/2a.png?raw=1"  class="center_ninety"/>


To allow Unity to know which Scenes are involved for run test and build, we need to add them to the **Build Setting**. Go to File >> Build Setting and add Scenes that are relevant to your game:

> From Unity Documentation: Select **Add Open Scenes** to add all currently open Scenes to the build. You can also drag Scene Assets from your **Project window**[](https://docs.unity3d.com/Manual/ProjectView.html)  
into this window. 

You should see something like this for example after adding a couple of Scenes:

<img src="https://www.dropbox.com/s/tto5ebkzpqj6ax1/1.png?raw=1"  class="center_ninety"/>

Some preparation before we proceed:

Move the script `GameManager.cs` to the **Root** GameManager GameObject, instead of having it attached to a child object like we did in the previous Lab:

<img src="https://www.dropbox.com/s/4dumikbkjkklb3e/3.png?raw=1"  class="center_ninety"/>


## C#: Properties
Open `GameManager.cs` and add the following lines:

```java

// Singleton Pattern
private  static  GameManager _instance;
// Getter
public  static  GameManager Instance
{
	get { return  _instance; }
}
```

We know from before that the `static` keyword allows us to gain access to the instance via the Class name, and there will only be one instance that can be linked to this variable `Instance` at any given time. This time round, we use `Properties` to allow access control to the variable, instead of having the usual `Public` fields. We can get properties very simply as shown above, and setting properties is also equally simple as shown in the example below:

```java
//Member variables can be referred to as fields.  
private  int _healthPoints; 

//healthPoints is a basic property  
public  int healthPoints { 
	get { 
		//Some other code  
		// ...
		return _healthPoints; 
	} 
	set { 
		// Some other code, check etc
		// ...
		_healthPoints = value; // value is the amount passed by the setter
	} 
}

// usage
Debug.Log(player.healthPoints); // this will call instructions under get{}
player.healthPoints += 20; // this will call instructions under set{}, where value is 20
```

> Optionally, you can have `private set` instead of just `set` to disallow other classes from setting it.

## The Singleton Pattern
Directly quoted straight from [here](https://wiki.unity3d.com/index.php/Singleton): *The [singleton](http://en.wikipedia.org/wiki/Singleton_pattern) pattern is a way to ensure a class has **only a single globally accessible instance** available at **ALL** times. Behaving much like a regular static class but with some advantages. This is very **useful for making global manager type** classes that hold global variables and functions that many other classes need to access.*

In the `Awake()` function of any script, add the following instructions:

```java
private  void  Awake()
{
	// check if the _instance is not this, means it's been set before, return
	if (_instance  !=  null  &&  _instance  !=  this)
	{
		Destroy(this.gameObject);
		return;
	}
	
	// otherwise, this is the first time this instance is created
	_instance  =  this;
	// add to preserve this object open scene loading
	DontDestroyOnLoad(this.gameObject); // only works on root gameObjects
}
```

> Note that if `this.gameObject` is **NOT root level** GameObject, **it will not work**. The editor will warn you: *DontDestroyOnLoad only work  for  root GameObjects or components on root GameObjects*.

Adding the above property and Singleton pattern will turn any gameObject to be persistent between screen changing. 

## Singleton Class
If you have several gameObjects to stay persistent, you can implement the above instructions on the relevant scripts, but this will result in many **boilerplate** code. To avoid this, we can create a dedicated script that can be inherited by any other scripts and turning them into Singletons. 

**Create** a new script called `Singleton.cs`, and declare the following properties:
> This code is obtained from [this site](http://wiki.unity3d.com/index.php/Singleton) with modifications

```java
using UnityEngine;

public  class Singleton<T> : MonoBehaviour  where  T : MonoBehaviour
{
	private  static  T _instance;
	public  static  T instance
	{
		get
		{
			return  _instance;
		}
	}

	public  virtual  void  Awake ()
	{
		Debug.Log("Singleton Awake called");

		if (_instance  ==  null) {
			_instance  =  this  as T;
			DontDestroyOnLoad (this.gameObject);
		} else {
			Destroy (gameObject);
		}
	}
}
```

The `virtual` method allows `override` by members inheriting this class, and the member can utilise this base `Singleton` class as such in a new script `GameUI.cs`:

```java
using UnityEngine;

public  class GameUI : Singleton<GameUI>
{
	override  public  void  Awake(){
		base.Awake();
		Debug.Log("awake called");
		// other instructions...
	}
}
```
Attach this script `GameUI.cs` in the UI GameObject, and you will observe that `UI` gameobject along with ALL its children it will be under `DontDestroyOnLoad` list at runtime. Modify `GameManager.cs` as well to adopt the Singleton Pattern.


<img src="https://www.dropbox.com/s/wctzufnf7hy1ik6/4.png?raw=1"  class="center_ninety"/>


Now suppose Mario reaches the end of the level (hits the castle door, you can download the castle prefab at the course handout or use your own), and would like to go to the next scene, we need to detect some Collision as such:

```csharp
using System.Collections;
using UnityEngine;

public  class ChangeScene : MonoBehaviour
{
	public  AudioSource changeSceneSound;
	void  OnTriggerEnter2D(Collider2D other)
	{
		if (other.tag  ==  "Player")
		{
			changeSceneSound.PlayOneShot(changeSceneSound.clip);
			StartCoroutine(LoadYourAsyncScene("MarioLevel2"));
		}
	}

	IEnumerator  LoadYourAsyncScene(string sceneName)
	{
		yield  return  new  WaitUntil(() =>  !changeSceneSound.isPlaying);
		CentralManager.centralManagerInstance.changeScene();
	}
}
```

And implement `changeScene` method in `CentralManager.cs` using `LoadSceneAsync`. There's more details on this API that can be found [here](https://docs.unity3d.com/ScriptReference/SceneManagement.SceneManager.LoadSceneAsync.html). Its Synchronous version can be found [here](https://docs.unity3d.com/ScriptReference/SceneManagement.SceneManager.LoadScene.html). 

```java
    public void changeScene()
    {
        StartCoroutine(LoadYourAsyncScene("MarioLevel2"));
    }


    IEnumerator LoadYourAsyncScene(string sceneName)
    {
        // The Application loads the Scene in the background as the current Scene runs.
        // This is particularly good for creating loading screens.
        AsyncOperation asyncLoad = SceneManager.LoadSceneAsync(sceneName, LoadSceneMode.Single);
        // Wait until the asynchronous scene fully loads
        while (!asyncLoad.isDone)
        {
            yield return null;
        }
    }
```
* Add your own Castle Door in the prefab as show, with `IsTrigger` property enabled
* Add your own change-scene sound effect 

<img src="https://www.dropbox.com/s/lz88io3bqd9rg3b/5.png?raw=1"  class="center_ninety"/>


Now test run and observe how you can load to the next scene preserving the **score** as well as PowerUp, whatever powerup that you have collected in the previous scene, if not yet consumed will be carried over to the next scene. 

## Controversy
The Singleton pattern has been somewhat [controversial](https://softwareengineering.stackexchange.com/questions/40373/so-singletons-are-bad-then-what/218322#218322), although at the end of the day it is up to the developer's wisdom on whether the pattern should be used. It is prone to abuse, and it is quite difficult to debug, for reasons laid out below.

### Testing Difficulty

Our second scene, `MarioLevel2` does not have any managers or UI system: the score text, the powerup icons, overlay panel and restart button (if you implement that), etc. This is because the UI GameObject, and the Managers GameObjects are instantiated in the first Scene, and carried over. They're under the `DontDestroyOnLoad` list:

<img src="https://www.dropbox.com/s/vwjuzze5t9et8y5/6.png?raw=1"  class="center_ninety"/>

This makes it **impossible** to test the correctness of the second scene *by itself*. You'd have to start from the first scene, and quickly dash to the end of the scene to load this second scene to test. It will be quite ridiculous to continue doing this if you have ten separate scenes and more. 

### Object Reference Bug
Another issue with Singletons is that **ANY references** in the singleton scripts has to be persistent across scenes as well. This is the reason why we made the UI gameobject along with all its children Singleton. Take a look at this example reference in `GameManager.cs`:

```java
public  Text score;

public void increaseScore(){
    Debug.Log("Player score is : " + playerScore);
    playerScore += 1;
    setScore();
    OnEnemyDeath();
}

public void setScore(){
    score.text = "SCORE: " + playerScore.ToString();
}
```

The reference `score` is probably linked up in the Editor in the first scene, as shown below:
<img src="https://www.dropbox.com/s/u34q6erzp0lc3g9/7.png?raw=1"  class="center_ninety"/>

We have to be **completely certain** that the reference `score` is **NOT obsolete** in the next scene. All gameobjects will be **destroyed** if we load new scene with the option `LoadSceneMode.Single` (unless we use `LoadSceneMode.Additive` but that will be pretty weird to just simply keep the objects alive if we aren't using both scenes in parallel). 

It is quite a chore to do this, especially if our project is large enough and we have too many Managers controlling different things in the Scene (and not just UI as shown in this simple Lab example). The usage of `events` definitely help though, so for instance `OnEnemyDeath();` will invoke any subscribers in any Scene -- the *current* enemy being killed in either Scene. 
