---
title: Designing a Combo System
permalink: /unity/unity_for_adults_2
key: unity-unity_for_adults_2
layout: article
nav_key: unitypro
sidebar:
  nav: unitypro
license: false
aside:
  toc: true
show_edit_on_github: false
show_date: false
---

# The Combo System
This suggested combo system works with Scriptable Object and a Manager Script meant to be attached on the player (that can cast the regular skill or the combo skill). As usual, we will make **each** skill and **each** combo a **Scriptable Object**, and **keep** **track** which Combo may potentially be casted given a current **key press**. 

## Skill Keys
The first step to do is to decide the keys that need to be pressed to activate *regular skills*, something that the character can keep casting on a regular basis. 

Create a new script called `SkillKeys.cs`:

```java
using UnityEngine;

[CreateAssetMenu(fileName = "SkillKeys", menuName = "ScriptableObjects/SkillKeys", order = 10)]
public class SkillKeys : ScriptableObject
{
    [Header("Inputs")]
    public KeyCode key;

    public bool isSameAs(SkillKeys k)
    {
        return key == k.key;
    }
}
```

## Regular Skills
Then, we also need to define the types of regular skill (the visual effect, sound effect, etc) that a character can cast. Create a new script called `RegularSkill.cs`:

```java
using UnityEngine;

public enum AttackType
{
    heavy = 0,
    light = 1,
    kick = 2,
    //test

}

[System.Serializable]
public class Effect
{
    // change this to your own data structure defining visual or audio effect of this skill
    public GameObject particleEffect;
    // o
}

[System.Serializable]
public class Attack
{
    public float length;

    // change this to your own data structure defining visual or audio effect of this skill
    public Effect effect;
}

[CreateAssetMenu(fileName = "RegularSkill", menuName = "ScriptableObjects/RegularSkill", order = 9)]
public class RegularSkill : ScriptableObject
{
    public AttackType skillType;
    public SkillKeys key;
    public Attack attack;
}
```


## Combo Skill
Finally, we need another script to describe our Combo: its type, effect, skill keys to trigger, and a simple *check* on whether the current given input is the correct key (**on track** to trigger the Combo):

```java
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

[System.Serializable]
public enum ComboType
{
    electric = 0,
    explosion = 1,
    shine = 2
};

[CreateAssetMenu(fileName = "ComboSkill", menuName = "ScriptableObjects/ComboSkill", order = 10)]
public class ComboSkill : ScriptableObject
{
    // a list of combo inputs that we will cycle through
    public List<SkillKeys> inputs;
    // public ComboAttack comboAttack; // once we got through all inputs, then we summon this attack
    public Attack attack;
    public UnityEvent onInputted;
    public ComboType comboType;

    int curInput = 0;

    public bool continueCombo(SkillKeys i)
    {
        if (currentComboInput().isSameAs(i))
        {
            curInput++;
            if (curInput >= inputs.Count) // finished the inputs and we should cast the combo
            {
                onInputted.Invoke();
                //restart the combo
                curInput = 0;
            }
            return true;
        }
        else
        {
            //reset combo
            ResetCombo();
            return false;
        }
    }

    public SkillKeys currentComboInput()
    {
        if (curInput > inputs.Count) return null;
        else return inputs[curInput];
    }

    public void ResetCombo()
    {
        curInput = 0;
    }
}
```

Note the instruction `onInputted.Invoke();`, means that in this design we expect some other callback method subscribed to this combo's `onInputted` event, which triggers whatever animation or effects necessary when the combo is successfully launched. 

Also, the method `continueCombo` keeps track of the input entered *so far*. Thus the instantiation of `ComboSkill` is *character dependent*, i.e: you need to instantiate one `ComboSkill` per character utilising it. 

> We aren't saying that this is the *best* solution. If you have other designs you deem better, you're free to use it in your Project. 


## Instantiation: SkillKey, RegularSkill, ComboSkill
Now create a few skill keys scriptable objects like the example shown:
<img src="https://www.dropbox.com/s/1um41ht12cd1tvr/1.png?raw=1"  class="center_ninety"/>

And a few regular skills scriptable objects that utilise a skill key:
<img src="https://www.dropbox.com/s/11owatje8uz4ij9/2.png?raw=1"  class="center_ninety"/>

Finally, a few combos utilising sequences of skill keys:
<img src="https://www.dropbox.com/s/ja7zivf37d9p2w5/3.png?raw=1"  class="center_ninety"/>


For this demo purpose, what we created was 3 different skill keys named:
- HeavyKey (A)
- KickKey (S)
- LightKey (D)

Each key will trigger a regular skill called:
- HeavyPunchSkill (utilising HeavyKey)
- KickSkill (utilising KickKey)
- LightPunchSkill (utilising LightKey)

A combination of a few keys will trigger a combo skill instead:
- HeavyKey x3: ElectricCombo
- KickKey x3:  ExplosiveCombo
- Heavy-Light-KickKey: MagicCombo 

You can name and set your own SkillKeys, RegularSkills, and ComboSkills as you like. You may also change the structure of the `effect` instance defined under `public class Attack` in `RegularSkill.cs`. Simply add more fields in the `Effect` class. For example, it is common to trigger animations too and you may add relevant Animator parameter names that need to be triggered upon casting this skill or combo, and use them later on using `<AttackInstance>.effect.<ParameterName>`. 

## The ComboManager

Now we need a script (to be attached to the Player gameobject) to utilize these RegularSkills and ComboSkills. Create a new script `ComboManager.cs` declaring the following setup variables:

```java
using System.Collections.Generic;
using UnityEngine;

public class ComboManager : ComboManagerBase
{
    [Header("Setup")]
    public float comboLeeway = 0.2f; // how fast should the delay between each key press be? 
    public List<RegularSkill> skills; // a list of ALL basic skills player can do
    public List<ComboSkill> combos; // a list of ALL combo we can do, each combo has an id based on its location in the list
}
```

In the inspector, we need to input the skills and combos that this player can do. Drag the respective scriptable objects we created earlier: 

<img src="https://www.dropbox.com/s/8tj5lmh2ah1rl1j/4.png?raw=1"  class="center_ninety"/>

Now on to the implementation of `ComboManager.cs`, Declare the following private variables:

```java
	// for visual effects
    private Animator animator;
    private Dictionary<AttackType, ParticleSystem> skillDictionary =
    new Dictionary<AttackType, ParticleSystem>();
    private Dictionary<ComboType, ParticleSystem> comboDictionary =
    new Dictionary<ComboType, ParticleSystem>();

	// logic
    private Attack curAttack = null; // currently executed attack, can be regular attack or combo attack
    private RegularSkill lastInput;
    private List<int> currentPossibleCombosID = new List<int>(); // keep track of id of the combos combos that could be accessed by the same current starting attack
    private float timer = 0; // to keep track how long current combo will play
    private bool skipFrame = false;
    private float currentComboLeeway; //to keep track time passed between each skill press that makes a combo
```

## C#: Dictionaries
Variables under `visual effects` will be instantiated under `Start()`. These variables hold the **references** to the relevant triggers so that we can indicate to the player that a combo or regular skill is currently happening. We chose to use *Dictionaries* in this example. Like in any other similar programming language, we can initialise Dictionaries in C# using:

`Dictionary<Key, Value> dictionary = new Dictionary<Key,Value>();` 

And then we can modify items in it or obtain `Value` given the `Key`:
```
dictionary.Add(newKey, newValue);
Value v = dictionary[inputKey];
bool success = dictionary.Remove(existingKey);
```

Our dictionary `skillDictionary` and `comboDictionary` `Value` field contains *references* to the particle system **component** that has to be triggered (`.Play()`) during runtime whenever this skill or combo is casted. The key to each of these references is the respective `AttackType` or  `ComboType` we defined earlier in `RegularSkill.cs` and `ComboSkill.cs`.

```java
    void Start()
    {
        animator = GetComponent<Animator>();
        InitializeCombosEffects();
        InitializeRegularSkillsEffects();
    }

    void InitializeCombosEffects()
    {
        // loop through the combos
        for (int i = 0; i < combos.Count; i++)
        {
            ComboSkill c = combos[i];

            // register callback for this combo on this manager
            c.onInputted.AddListener(() =>
            {
                // Call attack function with the combo's attack
                skipFrame = true; // skip a frame before we attack
                doComboAttack(c);
                ResetCurrentCombos();
            });
            // instantiate
            GameObject comboEffect = Instantiate(c.attack.effect.particleEffect, Vector3.zero, Quaternion.identity);
            comboEffect.transform.parent = this.transform; // make mario the parent
            // reset its local transform
            comboEffect.transform.localPosition = new Vector3(1, 0, 0);
            // add to particle system list
            comboDictionary.Add(c.comboType, comboEffect.GetComponent<ParticleSystem>());
        }
    }

    void InitializeRegularSkillsEffects()
    {
        // loop through regular skills effect
        for (int i = 0; i < skills.Count; i++)
        {
            RegularSkill r = skills[i];
            GameObject skillEffect = Instantiate(r.attack.effect.particleEffect, Vector3.zero, Quaternion.identity);
            skillEffect.transform.parent = this.transform; // make mario the parent
            // reset its local transform
            skillEffect.transform.localPosition = new Vector3(1, 0, 0);
            // add to particle system list
            skillDictionary.Add(r.skillType, skillEffect.GetComponent<ParticleSystem>());
        }
    }
```

We need three other helper function in `ComboManager.cs` before we can code its logic under `Update()`. First, is the callback when regular attack is casted:

```java
    void doRegularAttack(RegularSkill r)
    {
        animator.SetTrigger("CastBasic");
        // Attack(r.attack);
        curAttack = r.attack;
        timer = r.attack.length;
        // particle cast
        skillDictionary[r.skillType].Play();
    }
```

Second, is the callback when combo attack is casted:
```java
    void doComboAttack(ComboSkill c)
    {
        animator.SetTrigger("CastCombo");
        curAttack = c.attack;
        timer = c.attack.length;
        // particle cast
        comboDictionary[c.comboType].Play();
    }
```

Third, is a method to reset current *possible* combo list tracked by `ComboManager` and its combos:
```java
    void ResetCurrentCombos()
    {
        currentComboLeeway = 0;
        //loop through all current combos and reset each of them
        for (int i = 0; i < currentPossibleCombosID.Count; i++)
        {
            ComboSkill c = combos[currentPossibleCombosID[i]];
            c.ResetCombo();
        }
        currentPossibleCombosID.Clear();
    }
```

## C#: Abstract Class
Since it is imperative for the ComboManager to implement these three helper functions, and that these three functions will only by used by ComboManager and no one else, we can create an **abstract class** (instead of interface which requires the methods to be public because that's what *interface* is for). 

> Abstraction in C# is the process to **hide** the internal details and showing only the functionality. The **abstract modifier** indicates the incomplete implementation.

Create a new script, `ComboManagerBase.cs`:

```java
using UnityEngine;

public abstract class ComboManagerBase : MonoBehaviour
{
    protected abstract void doRegularAttack(RegularSkill r);
    protected abstract void doComboAttack(ComboSkill c);
    protected abstract void ResetCurrentCombos();
}
```

And inherit this in `ComboManager.cs`:
```java
public class ComboManager : ComboManagerBase
```

> Note that you can write **implementations** inside abstract classes (like regular methods, but not shown in this example). If we just want to declare the method, we can add the `abstract` keyword which means that it has no *body* or *implementation* and declared inside the abstract class only. Eventually, an abstract method **must** be implemented in all non-abstract classes inheriting it using the `override` keyword. 

Add the keywords: `protected override` in front of these three methods in `ComboManager.cs` to remove the errors. 

## Combo Manager Logic
Under `Update`, we need to constantly check if there's any last key press, or if the previous effect is playing, and determine if there's any possible combos with the current key press (if any). 

Firstly, we need to check if there's current attack that's being casted or playing. We quit immediately and this will be repeatedly call for as many frames that can be fit in `timer`, which contains the length of the currently playing attack. 

```java
        // if current attack is playing, we dont want to disturb it
        if (curAttack != null)
        {
            if (timer > 0) timer -= Time.deltaTime;
            else curAttack = null;
            return; // end it right here if there's a current attack playing
        }
```

Next, we need to check if there's already combos registered in `currentPossibleCombosID`. This array contains the **indexes** of combos that can possibly happen given input in the previous frames:
```java
        // if current combo is not empty, increase leeway count
        if (currentPossibleCombosID.Count > 0)
        {
            // increase leeway, this means we are waiting for the next sequence
            currentComboLeeway += Time.deltaTime;
            if (currentComboLeeway >= comboLeeway)
            {
                // if time's up, combo is not happening
                // cast last input if any 
                if (lastInput != null)
                {
                    doRegularAttack(lastInput);
                    lastInput = null;
                }
                ResetCurrentCombos();
            }
        }
        else
        {
            // no combos currently registered, reset leeway to ensure we don't have unclean old values
            currentComboLeeway = 0;
        }
```

The variable `currentComboLeeway` works as follows:
- Suppose you have a Combo X that requires you to press key A, then D, then C.  
- **If you've pressed A in the previous frame**, the **id** of Combo X would've existed inside `currentPossibleCombosID` (will implement this later). Now you *need to press D* within `comboLeeway` which value i set in the inspector. 
	- Currently, it is set at 0.2 seconds as example.
-  We continuously **increase** `currentComboLeeway` value and *if* it has passed 0.2s, no combo will be happening (means key D isn't pressed at all and time's up).
- We **cast** the **last** known regular attack (the attack invoked when we press A), and **reset** all combo-tracking variables in this script.

Now let's take care of the current input (if any),
```java
        RegularSkill input = null;
        // loop through current skills and see if the key pressed matches
        foreach (RegularSkill r in skills)
        {
            if (Input.GetKeyDown(r.key.key))
            {
                input = r;
                break;
            }
        }

		// return if there's no input currently that matches any skill
        if (input == null)
        {
            return;
        }
        
		// set current input as last known input
        lastInput = input;
```

If there's new input, loop through our current combos to see if it matches the *next input* of current possible combos. We also take note of the combo ID that will *never* happen because current input doesn't trigger that combo anymore (not the supposed next key to press).

```java
		List<int> remove = new List<int>();
        // loop through our current combos to see if it continues existing combos
        for (int i = 0; i < currentPossibleCombosID.Count; i++)
        {
            // get the actual combo from the combo ids stored in currentCombos
            ComboSkill c = combos[currentPossibleCombosID[i]];
            // if this input is the next thing to press
            if (c.continueCombo(lastInput.key))
            {
                currentComboLeeway = 0;
            }
            else
            {
                // this combo isn't happening, we need to remove it
                // take note of the combo id 
                remove.Add(currentPossibleCombosID[i]);
            }
        }
```

Then finally check if `skipFrame` turns `true` because any combo above is **activated**. This is possible because of the callback earlier in `InitializeCombosEffect`. This callback will happen if `continueCombo()` invokes it, means the current input is the last input i the Combo and will activate the combo. 

```java
        if (skipFrame)
        {
            skipFrame = false;
            return;
        }
```

If it doesn't return right above, we need to check if there's **new combos** that can be added to the `currentPossibleCombosID` due to current input key (if it is not already tracked):
```java
        // adding new combos to the currentCombo list with this current last known input 
        for (int i = 0; i < combos.Count; i++)
        {
            if (currentPossibleCombosID.Contains(i)) continue;

            // if it's not being checked already, attempt to add combos into current combos that START with this current last known input 
            if (combos[i].continueCombo(lastInput.key))
            {
                currentPossibleCombosID.Add(i);
                currentComboLeeway = 0;
            }
        }
```

The last two parts of the logic removes any existing combo in `currentPossibleCombosID` that will never happen because the sequence isn't obeyed anymore with current last known input:

```java
        // remove stale combos from current combos
        // recall 'remove' contains combo IDs to remove
        foreach (int i in remove)
        {
            currentPossibleCombosID.Remove(i);
        }
```

... and do basic attack if there's no possible combo that can happen anymore (`currentComboLeeway` expired or the current key doesn't start any combo):
```java
        // do basic attack if there's no combo
        if (currentPossibleCombosID.Count <= 0)
        {
            doRegularAttack(lastInput);
        }
```


The gif below shows an example of three regular skills, and three combos invoked. It might not be very clear in a gif, so refer to the lab recording if you are stuck. 

<img src="https://www.dropbox.com/s/ziuapq9bsx5l81n/combos.gif?raw=1"  class="center_ninety"/>
