---
title: Inventory System 
permalink: /unity/unity_for_teens_2
key: unity-unity_for_teens_2
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

# Inventory System
Another useful application of ScriptableObject is to utilise them as an **inventory system**. The idea is to have an Inventory Scriptable Object, and a Scriptable Object for every possible *item*. We can apply this idea to our Powerup system. 

Recall that in the previous lab, we created the UI for powerups in the Scene: 
> They were initially disabled so we won't see them until we collect the powerup. 

<img src="https://www.dropbox.com/s/oc5pplcwozru3b1/22.png?raw=1"  class="center_ninety"/>


We used the *actual instances* of the consumable GameObject (red or orange mushroom) to `cast` the effect on the Player's maxSpeed or jumpSpeed when the key z or x is pressed. This does not allow us to *carry over* the powerup effect to the next scene when used.

**This gif below shows the result when we used the Singleton implementation.** The redmushroom powerup, although it wasn't used yet, wasn't carried over in the second scene. Notice the RedMushroom(Clone) **exist** in the hierarchy despite being *unseen* because the gameobject has to be there if we want to cast the skill
> Bad design, we know. They were made like that initially for easy planning.

<img src="https://www.dropbox.com/s/vu1gfppmcw3y22q/powerupgone.gif?raw=1"  class="center_ninety"/>

With the new architecture utilising Scriptable Object Events, we can retain our powerup to the next scene, without having any object under DontDestroyOnLoad, hence effectively making **each Scene a clean slate**.

<img src="https://www.dropbox.com/s/cudcegnxt9q1ndf/powerupstay.gif?raw=1"  class="center_ninety"/>

In order to do this, we need to plan our *Inventory system* for our powerups. 

## Inventory ScriptableObject
**Create** a new script `Inventory.cs` that serves as a base class. There's a few methods to add or remove items from the inventory, and also a state that indicate whether a game is started for the first time or not.
```java
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[SerializeField]
public class Inventory<T> : ScriptableObject
{
    public bool gameStarted = false;
    public List<T> Items = new List<T>();

    public void Setup(int size){
        for (int i = 0; i < size; i++){
            Items.Add(default(T));
        }
    }

    public void Clear(){
        Items = new List<T>();
        gameStarted = false;
    }

    public void Add(T thing, int index)
    {
        if (index < Items.Count)
            Items[index] = thing;
    }

    public void Remove(int index)
    {
       if (index < Items.Count)
            Items[index] = default(T);
    }

    public T Get(int index){
        if (index < Items.Count){
            return Items[index];
        }
        else return default(T);
    }
}
```
> Note: T indicate a generic type. If T is a reference type, then default(T) means `null`.

Then **create** another Scriptable Object script `Powerup.cs` to define each Powerup in **general**. This serves as a base template for all kinds of powerup, so you need to think carefully all kinds of effects from all powerup. For this example, we can decide that a powerup can either affect player's speed, or jump speed (or both). 

> Add other effects and methods as you wish. 

```java
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu(fileName = "Powerup", menuName = "ScriptableObjects/Powerup", order = 5)]
public class Powerup : ScriptableObject
{
#if UNITY_EDITOR
    [Multiline]
    public string DeveloperDescription = "";
#endif
	// index in the UI
    public PowerupIndex index;
	// texture in the UI
    public Texture powerupTexture;
    
    // list of things any powerup can do
    public int aboluteSpeedBooster;
    public int absoluteJumpBooster;

	// effect of powerup
    public int duration;

    public List<int> Utilise(){
        return new List<int> {aboluteSpeedBooster, absoluteJumpBooster};
    }

    public void Reset(){
        aboluteSpeedBooster = 0;
        absoluteJumpBooster = 0;
    }

    public void Enhance(int speedBooster, int jumpBooster){
        aboluteSpeedBooster += speedBooster;
        absoluteJumpBooster += jumpBooster;
    }
}
```
`PowerupIndex` is an enum type, declared in `PowerupManagerEV.cs` script. For now, **create** this script and add the following code to silent the error in `Powerup.cs`:
```java
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public enum PowerupIndex
{
    ORANGEMUSHROOM = 0,
    REDMUSHROOM = 1
}
public class PowerupManagerEV : MonoBehaviour
{
}
```

Then **create another script** to utilise the above base Inventory, called `PowerupInventory.cs`. We don't have to implement anything, just simply inherit `Inventory` with `Powerup` as the object type in the inventory. 
```java
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu(fileName = "PowerupInventory", menuName = "ScriptableObjects/PowerupInventory", order = 6)]
public class PowerupInventory : Inventory<Powerup>
{
}
```

> Sanity check: you should have created 4 new scripts by now, `Inventory.cs`, `Powerup.cs`, `PowerupInventory.cs`, `PowerupManagerEV.cs`. 

## Create Powerup Scriptable Object for Each Type of Powerup

As the title said, go to your Assets folder, and create this powerup scriptable object. One example is as such:

<img src="https://www.dropbox.com/s/l7acflc9zh8aufc/23.png?raw=1"  class="center_ninety"/>

Notice how we store the metadata of this powerup in this scriptable object. You can easily create other powerups of your choice.


## Create PowerupInventory Scriptable Object 
It simply looks like this:
<img src="https://www.dropbox.com/s/5dpktr4fm8pwt3l/26a.png?raw=1"  class="center_ninety"/>

## Create Powerup Prefab
**Create a new Prefab** resembling the item that you can collect to obtain this powerup. We can reuse our Consumable Mushroom. If you forget, it is just a regular GameObject with BoxCollider2D, Rigidbody, Animator, and a script controlling its patrol location:

<img src="https://www.dropbox.com/s/vtsyrxigtdzzcey/24.png?raw=1"  class="center_ninety"/>

Now what we want to happen is to detect collision between this mushroom and the player, and this will add the powerup UI on the top left hand corner like the previous lab using ScriptableObject events. However this time round, we need to **have some sort of custom parameters** to indicate the effect of the powerup to the Powerup Manager. 

To do this, we cannot use the vanilla `UnityEvent`. We need to create our own event.

## UnityEvents with Parameter
We need to create a `PowerupEvent` together with the **matching** `PowerupEventListener` for this, not the previous `GameEvent` and `GameEventListener` which don't require any parameters.

Here's the script to modify UnityEvents with Parameter, simply declare a new class inheriting `UnityEvent`and add in the `<Powerup>` parameter. This script can be named as `PowerupEventListener.cs`
```java

using UnityEngine;
using UnityEngine.Events;


[System.Serializable]
public class CustomPowerupEvent : UnityEvent<Powerup>
{
}

public class PowerupEventListener : MonoBehaviour
{
    public PowerupEvent Event;
    public CustomPowerupEvent Response;
    private void OnEnable()
    {
        Event.RegisterListener(this);
    }

    private void OnDisable()
    {
        Event.UnregisterListener(this);
    }

    public void OnEventRaised(Powerup p)
    {
        Response.Invoke(p);
    }
}
```

And here's the matching `PowerupEvent.cs` ScriptableObject:
```java
using System.Collections.Generic;
using UnityEngine;


[CreateAssetMenu(fileName = "PowerupEvent", menuName = "ScriptableObjects/PowerupEvent", order = 3)]
public class PowerupEvent : ScriptableObject
{
    private readonly List<PowerupEventListener> eventListeners = 
        new List<PowerupEventListener>();

    public void Raise(Powerup p)
    {
        for(int i = eventListeners.Count -1; i >= 0; i--)
            eventListeners[i].OnEventRaised(p);
    }

    public void RegisterListener(PowerupEventListener listener)
    {
        if (!eventListeners.Contains(listener))
            eventListeners.Add(listener);
    }

    public void UnregisterListener(PowerupEventListener listener)
    {
        if (eventListeners.Contains(listener))
            eventListeners.Remove(listener);
    }
}
```

Then, create a `OnPowerupCollected` Scriptable Object PowerupEvent **instance** in your Assets. 

Finally, **create a script** `ConsumableTriggerChecker.cs` to be attached to the mushroom, triggering this **CustomPowerupEvent event with `Powerup stats` passed as parameter:**
```java
using System.Collections;
using UnityEngine;
using UnityEngine.Events;

public class ConsumableTriggerChecker : MonoBehaviour
{
    public Powerup stats;
    public CustomPowerupEvent onCollected;

    void OnCollisionEnter2D(Collision2D col)
    {
        if (col.gameObject.CompareTag("Player"))
        {
            onCollected.Invoke(stats);
			Destroy(this.gameObject);
        }
    }
}
```

We can set up the inspector to be:
<img src="https://www.dropbox.com/s/r7abz7uxdkm2tqe/25.png?raw=1"  class="center_ninety"/>

## Powerup Listener
Now the final job is to implement the gameObject that acts as the listener to this `OnPowerupCollected` event. 

**Create** a root gameobject PowerupManager, and attach the previously created `PowerupManagerEV.cs` script to it. 

Add a few more instructions inside it:
* References to all ScriptableObjects containing Powerup-affected player values such as max speed and jump speed 
* Reference to current PowerupInventory 
* References to the Scene's Powerup Slots GameObjects
```java
public class PowerupManagerEV : MonoBehaviour
{
    // reference of all player stats affected
    public IntVariable marioJumpSpeed;
    public IntVariable marioMaxSpeed;
    public PowerupInventory powerupInventory;
    public List<GameObject> powerupIcons;

    void Start()
    {
        if (!powerupInventory.gameStarted)
        {
            powerupInventory.gameStarted = true;
            powerupInventory.Setup(powerupIcons.Count);
            resetPowerup();
        }
        else
        {
            // re-render the contents of the powerup from the previous time
            for (int i = 0; i < powerupInventory.Items.Count; i++)
            {
                Powerup p = powerupInventory.Get(i);
                if (p != null)
                {
                    AddPowerupUI(i, p.powerupTexture);
                }
            }
        }
    }
    
    public void resetPowerup()
    {
        for (int i = 0; i < powerupIcons.Count; i++)
        {
            powerupIcons[i].SetActive(false);
        }
    }
    
    void AddPowerupUI(int index, Texture t)
    {
        powerupIcons[index].GetComponent<RawImage>().texture = t;
        powerupIcons[index].SetActive(true);
    }

    public void AddPowerup(Powerup p)
    {
        powerupInventory.Add(p, (int)p.index);
        AddPowerupUI((int)p.index, p.powerupTexture);
    }

    public void OnApplicationQuit()
    {
        ResetValues();
    }
 }
```
The `Start` method contains a quick check on whether the game is started the first time or not. If not, we shall render whatever powerup exists in our inventory. 

The `AddPowerup` method is the callback for `PowerupEventListener` script we are going to attach on this PowerUpManager GameObject. **Set everything up in your scene as follows:**

<img src="https://www.dropbox.com/s/3ox1b3lyq0xzj18/27.png?raw=1"  class="center_ninety"/>

To summarise, here's the relationship between `OnPowerupCollected` Powerup Event and the Listener, and the update of Powerup UI (Slot 1 and 2):
<img src="https://www.dropbox.com/s/g39abxm6npalsv5/28a.png?raw=1"  class="center_ninety"/>

Since the powerup collected is now **added to the PowerupInventory**, the **content** of PowerupInventory will persist between Scene changes. Transform this `PowerupManager` gameobject as a prefab and instantiate it in the Second Scene, linking the necessary UI elements in that new Scene:

<img src="https://www.dropbox.com/s/x26yr0fz3m2e0n7/29.png?raw=1"  class="center_ninety"/>

The check at `Start()` method of `PowerupControllerEV.cs` will set the PowerupUI according to the content of `PowerupInventory` regardless on which Scene we are on. The contents of `PowerupInventory` ScriptableObject instance will be reset on each application quit.

We shall also reset PowerupInventory's content on player's death. Simply add the GameEventListener component to the PowerupManager Gameobject prefab:

<img src="https://www.dropbox.com/s/qpl7dr69h9oz8tv/30.png?raw=1"  class="center_ninety"/>

Test run and obtain some powerup in the first scene. When you proceed to the next scene, you should still have the powerup with you. 
