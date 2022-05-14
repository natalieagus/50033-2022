---
title: Scriptable Object Game Architecture
permalink: /unity/unity_for_teens_1
key: unity-unity_for_teens_1
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

# Game Architecture With Scriptable Object
Ultimately, if you choose to go down the Singleton Path, the choice is entirely up to you but we feel that it is our responsibility to introduce you to another great alternative using Scriptable Objects. This will revamp our existing project by quite a lot, but the benefits are worth it:
* Scenes are clean slates
* No dependency between Systems and they are modular
* Prefabs work on their own
* *Pluggable* custom components

This [talk](https://www.youtube.com/watch?v=raQ3iHhE_Kk) inspires the existence of this section. We simply do not have enough time (unfortunately) to go into every single detailed implementation of common concepts such as game inventory, skill tree, etc but we hope that this quick introduction will point you into the right direction in the future. 

## Preparation

We need two Scenes with completely *clean slate*. That's right. **Clean Slates**. We can't reuse any of these Scripts anymore: `EnemyController`, `GameManager`, `SpawnManager`, `PowerupManager`, `Consume`, `PlayerController`, among others because we are changing the whole architecture. 

To get you up to speed, you can **create** a new Scene and name it `MarioLevel1`:

<img src="https://www.dropbox.com/s/f1e0kzgv0yjstsr/8.png?raw=1"  class="center_ninety"/>

* Remove any scripts from `Mario` 
* You can leave EnemySpawnPool gameobject with `ObjectPooler.cs` attached. Set the `Items To Pool` to 0. 
* The rest of the stuffs that can be in the Scene is simply stuffs with **no script at all**: just the Brick, some Pipe, and the Castle. 

**Create another Scen**e `MarioLevel2` as such:

<img src="https://www.dropbox.com/s/ejqnlh7udzy74u0/9.png?raw=1"  class="center_ninety"/>

Leave Mario out for now. 

For the UI on both Scenes, simply have a ScoreText placed on the upper right hand corner like we did in the first lab. 

## Using Scriptable Objects as Reference

Previously, to control Mario we have public variables in `PlayerController`:
```java
    public float jumpSpeed;
    public float maxSpeed;
```
that we can set in the Inspector to get the right feel on Mario's max speed and jump speed. We also have implemented **powerups**, that changes directly these values when the powerup is consumed, something like this:
```java
    public void consumedBy(GameObject player){
        // give player jump boost
        player.GetComponent<PlayerController>().jumpSpeed += 10;
        StartCoroutine(removeEffect(player));
    }

    IEnumerator removeEffect(GameObject player){
        yield return new WaitForSeconds(5.0f);
        player.GetComponent<PlayerController>().upSpeed -= 10;
    }
```

Allowing a direct interaction between script, and passing references around like this is not a good idea. For prototyping, *sure, maybe just or a quick proof of concept*. We can also use Managers to aid interaction between scripts, and usually these managers have to be made **Singletons**. As discussed, this potentially bring about bugs if the Singleton script refers to Scene-dependent instances. 

A better way is to decouple their interaction:

<img src="https://www.dropbox.com/s/f5835k8m07kzey9/10.png?raw=1"  class="center_ninety"/>

The nodes in black denote ScriptableObjects instances. The idea is to store Mario's max speed and jump speed variable somewhere else, and then refer to it at runtime. Powerup controller can also modify these values at runtime, and since Player controller are reading from these data containers, the changes will be reflected at runtime. 

### C#: Method Overloading
**Create a script** called `IntVariable.cs`:

```java
using UnityEngine;

[CreateAssetMenu(fileName = "IntVariable", menuName = "ScriptableObjects/IntVariable", order = 2)]
public class IntVariable : ScriptableObject
{
#if UNITY_EDITOR
    [Multiline]
    public string DeveloperDescription = "";
#endif
    private int _value = 0;
    public int Value{
        get{
            return _value;
        }
    }

    public void SetValue(int value)
    {
        _value = value;
    }

    // overload
    public void SetValue(IntVariable value)
    {
        _value = value._value;
    }

    public void ApplyChange(int amount)
    {
        _value += amount;
    }

    public void ApplyChange(IntVariable amount)
    {
        _value += amount._value;
    }
}

```
* You can write helper functions in the ScriptableObject
* You can also write handy description in the Inspector when you instantiate this Object
* Notice the method **overloading** for `SetValue` and `ApplyChange`

Afterwards, create two instances of these `IntVariable` ScriptableObject by right clicking in Project tab >> Create >> ScriptableObjects >> IntVariable and name them:
* MarioJumpSpeed
* MarioMaxSpeed

In `GameConstants.cs` that you have created before, add these two fields:
```java
    // Mario basic starting values
    public int playerStartingMaxSpeed = 5;
    public int playerMaxJumpSpeed = 30;
    public int playerDefaultForce = 150;
```

Then, **create a new script** named: `PlayerControllerEV.cs`  to control Mario, with the following declarations:
```java
using System.Collections;
using UnityEngine;


public class PlayerControllerEV : MonoBehaviour
{
    private float force;
    public IntVariable marioUpSpeed;
    public IntVariable marioMaxSpeed;
    public GameConstants gameConstants;
	  
	// other components and interal state
}
```

As for the components and internal Mario state, you're free to modify them as you wish, according to however you implement Mario's movement earlier. 

Then in the `Start` method, set the values based on what's set in `gameConstants`, our go-to reference for anything that is invariant in the game. 

```java
        marioUpSpeed.SetValue(gameConstants.playerMaxJumpSpeed);
        marioMaxSpeed.SetValue(gameConstants.playerMaxSpeed);
        force = gameConstants.playerDefaultForce;
```

Then simply utilise `marioUpSpeed` and `marioMaxSpeed` accordingly in your game. This is a sample implementation in `FixedUpdate`:
```java
    void FixedUpdate()
    {
        if (!isDead)
        {
            //check if a or d is pressed currently
            if (!isADKeyUp)
            {
                float direction = faceRightState ? 1.0f : -1.0f;
                Vector2 movement = new Vector2(force * direction, 0);
                if (marioBody.velocity.magnitude < marioMaxSpeed.Value)
                    marioBody.AddForce(movement);
            }

            if (!isSpacebarUp && onGroundState)
            {
                marioBody.AddForce(Vector2.up * marioUpSpeed.Value, ForceMode2D.Impulse);
                onGroundState = false;
                // part 2
                marioAnimator.SetBool("onGround", onGroundState);
                countScoreState = true; //check if Gomba is underneath
            }
        }
```

Bool `isDead`, `isADKeyUp`, and `isSpacebarUp` is set at `Update()` using `GetKeyUp` or `GetKeyDown` accordingly since these APIs are refreshed at each frame, and hence more suitable to be called in `Update` rather than `FixedUpdate`.

Test run and you should be able to control your Mario around as per normal: move left, right, and  jump. 

## Scriptable Objects Event System

Now **create a new script** called `EnemyControllerEV.cs`, and paste the code from `EnemyController.cs` we completed in the previous lab. **Delete any instructions** regarding `CentralManager` or `GameManager`. We will not be using any Singleton Managers right now. 

Our logic stays the same: if the enemy prefab detects collision with Mario:
* If Mario "lands" on it from above, then some `OnEnemyDeath` event must be invoked to trigger:
	* Creation of one more new enemy
	* Increment of score
* Else, some `OnPlayerDeath` event must be invoked to trigger:
	* Enemy's winning dance
	* Player death animation 
	* Probably a panel + restart button to let players repeat the stage

Since we are not using any GameManagers, the event system created relies entirely on ScriptableObjects. 

We need to create a `GameEvent` object and its `Listeners`. **Create a new script** called `GameEvent.cs`. This Scriptable Object-based event will store a list if `GameEventListeners`, and `Raise` them whenever the event is invoked. 

```java
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu(fileName = "GameEvent", menuName = "ScriptableObjects/GameEvent", order = 3)]
public class GameEvent : ScriptableObject
{
    private readonly List<GameEventListener> eventListeners = 
        new List<GameEventListener>();

    public void Raise()
    {
        for(int i = eventListeners.Count -1; i >= 0; i--)
            eventListeners[i].OnEventRaised();
    }

    public void RegisterListener(GameEventListener listener)
    {
        if (!eventListeners.Contains(listener))
            eventListeners.Add(listener);
    }

    public void UnregisterListener(GameEventListener listener)
    {
        if (eventListeners.Contains(listener))
            eventListeners.Remove(listener);
    }
}

```

Naturally, **create a script** called `GameEventListener.cs` that can be attached to any GameObject meant to subscribe to the `GameEvent`:
```java

using UnityEngine;
using UnityEngine.Events;

public class GameEventListener : MonoBehaviour
{
    public GameEvent Event;

    public UnityEvent Response;

    private void OnEnable()
    {
        Event.RegisterListener(this);
    }

    private void OnDisable()
    {
        Event.UnregisterListener(this);
    }

    public void OnEventRaised()
    {
        Response.Invoke();
    }
}
```

To test this concept, create **two** GameEvent ScriptableObjects called `OnPlayerDeath` and `OnEnemyDeath` in your Assets. It is **strongly suggested** that you're highly organised for this, creating folders with meaningful names as we will be creating quite a lot of events later on.

Then, in `EnemyControllerEV.cs`, declare the two `Unity Event` variables. Don't forget to include the Events library: 

```java
using UnityEngine.Events;

... 
    // events to subscribe
    public UnityEvent onPlayerDeath;
    public UnityEvent onEnemyDeath;
```

As usual, invoke the events when there's collision between the enemy and the player:
```java
    void OnTriggerEnter2D(Collider2D other)
    {
        // check if it collides with Mario
        if (other.gameObject.tag == "Player")
        {
            // check if collides on top
            float yoffset = (other.transform.position.y - this.transform.position.y);
            if (yoffset > 0.75f)
            {
                enemyAudioSource.PlayOneShot(enemyAudioSource.clip);
                KillSelf();
                onEnemyDeath.Invoke();
            }
            else
            {
                // hurt player
                onPlayerDeath.Invoke();
            }
        }
    }
```
## UnityEvent.Invoke
What does `Invoke` do? Well, as the name said it invoke **all registered callbacks** (runtime and persistent). Who are these items? That, is set on the **Inspector**, which is none other than the `Raised` methods of our ScriptableObject `OnPlayerDeath` and `OnEnemyDeath`. 

Attach the script into a **new Gomba prefab** (pretty much the same as before, just with a new script and name it differently so you can look back and differentiate between the previous version without scriptable object event system):

<img src="https://www.dropbox.com/s/gtke683pef2mfhh/11.png?raw=1"  class="center_ninety"/>

* Load our previously created `OnPlayerDeath` and `OnEnemyDeath` Scriptable Object instances
* On the right side, select the Raise() method as the **registered callback** when `Invoke()` is called. 


## UnityEvent Callback Signature
**Now who are the Listeners (subscribers) to these events?** We haven't set any subscribers yet. One obvious subscriber **is the enemy itself**, it is supposed to do the *victory dance* upon player's death. We can call the method directly after  `onPlayerDeath.Invoke();` instruction or we can be fancy and put the `GameEventListener` component into use. Add the **callback** for the response in `EnemyControllerEV.cs`:

```java
	// callbacks must be PUBLIC
    public void PlayerDeathResponse()
    {
        GetComponent<Animator>().SetBool("playerIsDead", true);
        velocity = Vector3.zero;
    }
```

We learned before that the **signature** of an Event must match the delegate. Since we are using **UnityEvent** instead of our own custom event, the **default signature** will be to have **no argument at all**. Notice that `Raise()` and `PlayerDeathResponse()` both have these signatures. 

> It is also important to notice that **Events DO NOT have return values**, because it simply doesn't make sense. Who do we return the values to? The Invoker? What if there's many subscribers, do we collect all return values? It doesn't make sense. 

Now to simply set this **callback** `PlayerDeathResponse` as the method called whenever `onPlayerDeath.Invoke();` is called, we attach the `GameEventListener` component for the Enemy gomba prefab, **set** `OnPlayerDeath` event as the event to listen to and **set** the response **Object** and **Method** at the inspector directly: 

<img src="https://www.dropbox.com/s/47fdg86j477ssbb/12.png?raw=1"  class="center_ninety"/>
 
Another listener to this `OnPlayerDeath` event should be the Player. Add the following method in `PlayerControllerEV.cs` as the callback (notice the same argument-less signature):

```java
    public void PlayerDiesSequence()
    {
        isDead = true;
        marioAnimator.SetBool("isDead", true);
        GetComponent<Collider2D>().enabled = false;
        marioBody.AddForce(Vector3.up * 30, ForceMode2D.Impulse);
        marioBody.gravityScale = 30;
        StartCoroutine(dead());
    }
    
    IEnumerator dead()
    {
        yield return new WaitForSeconds(1.0f);
        marioBody.bodyType = RigidbodyType2D.Static;
    }
```
> You're free to implement whatever Mario should do when he died, in the example above we have set the Animator to play some mario_death animation. 

Then similarly, attach another GameEventListener component in the Player gameobject, and link the method as the Response() and the `OnPlayerDeath` event as the event to listen to:

<img src="https://www.dropbox.com/s/cw6bt5m0tuas7w7/13.png?raw=1"  class="center_ninety"/>

Spawn a few enemy prefabs in the Scene to test. You can use the ObjectPooler if you want. Set the gameObject with `ObjectPooler` script attach to spawn the new prefabs:

<img src="https://www.dropbox.com/s/y4a6nlfr5dyacel/14.png?raw=1"  class="center_ninety"/>

Then to utilise the pool, **create a new script** called `SpawnManagerEV.cs` with pretty much the same code as we have seen before, but without any references to Managers:

```java
using UnityEngine;
using UnityEngine.SceneManagement;

public class SpawnManagerEV : MonoBehaviour
{
    public GameConstants gameConstants;
    void Start()
    {
        Debug.Log("Spawnmanager start");
        for (int j = 0; j < 2; j++)
            spawnFromPooler(ObjectType.greenEnemy);

    }

    void startSpawn(Scene scene, LoadSceneMode mode)
    {
        for (int j = 0; j < 2; j++)
            spawnFromPooler(ObjectType.gombaEnemy);
    }


    void spawnFromPooler(ObjectType i)
    {
        GameObject item = ObjectPooler.SharedInstance.GetPooledObject(i);

        if (item != null)
        {
            //set position
            item.transform.localScale = new Vector3(1, 1, 1);
            item.transform.position = new Vector3(Random.Range(-4.5f, 4.5f), gameConstants.groundDistance + item.GetComponent<SpriteRenderer>().bounds.extents.y, 0);
            item.SetActive(true);
        }
        else
        {
            Debug.Log("not enough items in the pool!");
        }
    }

    public void spawnNewEnemy()
    {
        ObjectType i = Random.Range(0, 2) == 0 ? ObjectType.gombaEnemy : ObjectType.greenEnemy;
        spawnFromPooler(i);
    }

}
```
where `gameConstants.groundDistance` is simply the offset required to place the enemy nicely above ground declared in `GameConstants.cs`:
```java
// use your own value, it might not be -1.0 for your case
public  float groundDistance =  -1.0f;
```

**Create a root GameObject** where we can attach this `SpawnManagerEV` script. Recall that as part of the previous checkoff, you need to spawn one new enemy when an enemy died. Now this can be done **very conveniently** by attaching a `GameEventListener` component to this gameobject, and attach the `spawnNewEnemy()` as the response + the `OnEnemyDeath` event as the event to listen to:

<img src="https://www.dropbox.com/s/hy7qegmc9e86vbr/15.png?raw=1"  class="center_ninety"/>

Now when Mario hits the Enemy, he died and the Enemy animates. When Mario kills the enemy, a new enemy is spawned. Hopefully you can see how **convenient** it is to use Scriptable Object Event Based System. When there's a need to add new subscriber, simply:
* Write a callback with the correct signature 
* Attach the Listener script to the gameobject 
* Set the appropriate event to listen to and the callback as the Response


The figure below summarises the relationship between the listeners and the events:

<img src="https://www.dropbox.com/s/3b3d3jkw97y5hgo/16.png?raw=1"  class="center_ninety"/>

## Persistent Scoring System

Updating score is fairly easy with this architecture. Suppose we want to add the Player's score every time the enemy is stomped by Mario (every time `OnEnemyDeath` is Raised), we can lay out a plan like this:

<img src="https://www.dropbox.com/s/yvtyl73ndk34g2u/17.png?raw=1"  class="center_ninety"/>

We can subscribe more than one responses in the GameEventListener, and each method will be called **in sequence**, that is `ApplyChange` first and then `UpdateScore`. 

**Create** a new Scriptable Object IntVariable **instance** and name it MarioScore. This will hold Mario's current score. 

Then, **create a script** called `ScoreMonitor.cs` and attach it to the Score Text Gameobject. This component monitors the value contained in `marioScore`.

```java
using UnityEngine.UI;
using UnityEngine;

public class ScoreMonitor : MonoBehaviour
{
    public IntVariable marioScore;
    public Text text;
    public void UpdateScore()
    {
        text.text = "Score: " + marioScore.Value.ToString();
    }
}
```

Then create a **root GameObject** called ScoreEventListener, with a GameEventListener component attached. Set up two responses for `OnEnemyDeath` as follows:

<img src="https://www.dropbox.com/s/bxzrcrse0qdr7d8/18.png?raw=1"  class="center_ninety"/>

## Resetting the Score
We probably want to clear player score back to zero `OnPlayerDeath` event, so we can add another Game Event Listener Script and set the Response to modify `MarioScore` directly as follows:

<img src="https://www.dropbox.com/s/exhoexx3hca1bkz/19.png?raw=1"  class="center_ninety"/>

Another event where we want to reset the score is when **application quits**. **Create** a new ScriptableObject GameEvent instance named `OnApplicationExit` for this, and a simple `GameManager.cs` Script to do basic stuffs like invoking this event when application exits:

```java
using UnityEngine;
using UnityEngine.Events;
using UnityEngine.SceneManagement;

public class GameManagerEV : MonoBehaviour
{
    public UnityEvent onApplicationExit;
	void OnApplicationQuit()
    {
        onApplicationExit.Invoke();
    }
}
```
You can create a root gameobject with this script, acting as a simple current-Scene-only Game Manager. Set the `OnApplicationExit` scriptable object as the unity event `onApplicationExit` field in the inspector. Then, head to **ScoreEventListener** GameObject and add yet another listener that resets Mario's score upon application exit:

<img src="https://www.dropbox.com/s/sxiejq92y5l1m1l/20.png?raw=1"  class="center_ninety"/>

Test run by increasing the score (stomp on some monsters) and then stopping the application. Press play again and ensure that the scoreText is back at zero. 

## Updating ScoreText when Scene starts
We only have one Scene for now, so the value of the score printed will always be `Score: 0` as set in the **scene**, but when we load the second scene later on, this will not work well. We need to update the ScoreText based on whatever the content of `MarioScore` is when Scene starts. 

This can be  done naturally in `ScoreMonitor.cs` by implementing the regular `Start` callback:
```java
public void Start()
{
    UpdateScore();
}
```

Obviously, we don't need the `OnApplicationQuit` game event, since we can just set Mario's score into zero in `ScoreMonitor` by implementing something like:
```java
void OnApplicationQuit(){
	marioScore.SetValue(0);
}
```
... but it will be pretty weird to set `marioScore` with a score monitor. 

## Changing Scenes 
We previously tried hard to preserve the value of Score using Singleton GameManager and Singleton UI (along with their children). Fortunately with this new architecture, we no longer need to do so. Our score inherently persistent inside `MarioScore` ScriptableObject, we just need to reset them accordingly when application quits or when Mario dies as we did above. 

**Create a new script** `ChangeSceneEV.cs` to be attached to the Castle Door GameObject with a BoxCollider2D (isTrigger checked) and AudioSource (optional):
```java
using System.Collections;
using UnityEngine;
using UnityEngine.SceneManagement;
public class ChangeSceneEV : MonoBehaviour
{
    public AudioSource changeSceneSound;
    void OnTriggerEnter2D(Collider2D other)
    {
        if (other.tag == "Player")
        {
            changeSceneSound.PlayOneShot(changeSceneSound.clip);
            StartCoroutine(ChangeScene("MarioGameEVLevel2"));
        }
    }

    IEnumerator WaitSoundClip(string sceneName)
    {
        yield return new WaitUntil(() => !changeSceneSound.isPlaying);
        StartCoroutine(ChangeScene("MarioGameEVLevel2"));

    }
    IEnumerator ChangeScene(string sceneName)
    {
        AsyncOperation asyncLoad = SceneManager.LoadSceneAsync(sceneName, LoadSceneMode.Single);
        // Wait until the asynchronous scene fully loads
        while (!asyncLoad.isDone)
        {
            yield return null;
        }
    }
}
```

<img src="https://www.dropbox.com/s/zhcw97xi3lblavr/21.png?raw=1"  class="center_ninety"/>

Now store `GameManager`, `ScoreEventListener`, `UI` (or whatever gameobject containing that `ScoreText`) and `Mario` as a **prefab**. Then place them in the second scene. Make sure to set up whatever you need in each GameObject inspector properly. 

Here's a gif to aid you. Ignore the PowerupManager for now. 
<img src="https://www.dropbox.com/s/sty9xmua42jl9nr/settings.gif?raw=1"  class="center_ninety"/>

Make sure both scenes are added to the BuildSetting. Open the first Scene, and test the transition. You should see that the score persists in the second scene. 
<img src="https://www.dropbox.com/s/251ipxt56bintyv/score.gif?raw=1"  class="center_ninety"/>

You can also *test* the second Scene **independently** now. **With this new architecture, each Scene is a clean slate.** 
