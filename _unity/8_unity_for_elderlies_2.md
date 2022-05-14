---
title: Creating States, Actions, and Decisions Scriptable Objects
permalink: /unity/unity_for_elderlies_2
key: unity-unity_for_elderlies_2
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

All Scriptable Object instances can be found under ScriptableObjects folder. For Actions and Decisions, they're pretty straightforward: one instance per script type with no other setup required:

<img src="https://www.dropbox.com/s/w5mfy3zp469m3ef/3.png?raw=1"  class="center_ninety"/>

The States instances however require more setup. For instance, click on ScriptableObjects >> States >> AITankChaserStates >> **PatrolChaser**, and you shall see this at the inspector:
<img src="https://www.dropbox.com/s/gl17dk7fnbw7w84/4.png?raw=1"  class="center_ninety"/>

**RemainInstate** is a special state with empty Actions and Transitions. It's effect is to stay at the same state and perform `DoActions` and `CheckTransitions` of the **same state** again at the **next** `Update()` call. 

**PatrolChaser** is the one of the States to be plugged into AITankChaser (the not-so-fun one). Click on other States and study its structure: a state can have multiple Actions and Transitions -- to be computed in *sequential* order as per `DoActions` and `CheckTransitions` methods in `State.cs`. 

Note that CheckTransitions will always call StateController's TransitionToState based on the value of decisionSucceeded:
```java
// In State.cs
	private void CheckTransitions(StateController controller)
	{
		controller.transitionStateChanged = false; //reset 
		for (int i = 0; i < transitions.Length; ++i)
		{
			//check if the previous transition has caused a change. If yes, stop. Let Update() in StateController run again in the next state. 
			if (controller.transitionStateChanged){
				break;
			}
			bool decisionSucceded = transitions[i].decision.Decide(controller);
			if (decisionSucceded)
			{
				controller.TransitionToState(transitions[i].trueState);
			}
			else
			{
				controller.TransitionToState(transitions[i].falseState);
			}
		}
	}
```
 It will only proceed to the next `i` **if** `transitions[i]` true or false state  is `RemainInState`:
```java
// in StateController.cs
	public void TransitionToState(State nextState)
	{
		if (nextState == remainState) return;
		currentState = nextState;
		transitionStateChanged = true;

		if(currentState.name == "ChaseScanner" || currentState.name == "ChaseChaser")gameObject.GetComponent<Animation>().Play();

		OnExitState();
	}
```

## AITankChaser
The Red AI Tank's (chaser tank) state transition diagram can be drawn such:
<img src="https://www.dropbox.com/s/1yn8qtxrgcfzh8r/5.png?raw=1"  class="center_ninety"/>

**What this Bot does in a nutshell:**
1.  Begin at state  `PatrolChaser` and perform `PatrolAction` of circulating around waypoint 1, 2, then 3 in that order. Afterwards, always perform `LookDecision` : cast a ray and see if it hits anybody with a tag of Player.
	- If `true` (yes), set the chase target as the Player object and store its location, and go to `ChaseChaser` State
	- If `false` (no), `RemainInState`: `PatrolState`  and **repeat** `PatrolAction` and `LookDecision` in the next update call
2. If the bot arrives at `ChaseChaser` State, perform **two** actions. 
	- Action 1: `ChaseAction` Move to the last chase target location of Player, and then 
	- Action 2: `AttackAction` — fire bullet **if** player is within range, otherwise this attack motion does nothing.
	- Then, to perform Decision called `ActiveStateDecision` that checks if PlayerTank gameobject is still **Active**:
		- If PlayerTank is **still active**: The tank will `RemainInState`: `ChaseChaser`   and **repeat** `ChaseAction` and `AttackAction` , followed by  `ActiveStateDecision` until the PlayerTank *dies* (inactive in Hierarchy). 
		- If PlayerTank is **no longer active**: — means *dead* — The bot will go back to `PatrolChaser` State.

The green state "Patrol" above corresponds to the Scriptable Object `PatrolChaser`:
<img src="https://www.dropbox.com/s/gl17dk7fnbw7w84/4.png?raw=1"  class="center_ninety"/>


and the red state "Chase" in the diagram above corresponds to the Scriptable Object `ChaseChaser` in the Project.
<img src="https://www.dropbox.com/s/mm6vntc2yz8l1xb/6a.png?raw=1"  class="center_ninety"/>

Hence the StateController component in AITankChaser is set to start at `PatrolChaser` as it's **starting state**:

<img src="https://www.dropbox.com/s/78ssyl0uaspu19j/7a.png?raw=1"  class="center_ninety"/>

## AITankScanner
The Green AI Tank's (scanner tank) state transition diagram can be drawn such:

<img src="https://www.dropbox.com/s/4wwsqcjctjdsift/8.png?raw=1"  class="center_ninety"/>

**What this Bot does in in a nutshell:**
1.  An AITankScanner will start in the `PatrolScanner`  State,
2.  Similarly, it performs the action of going around waypoints if there’s no player in sight (`PatrolAction`).
3.  It will also perform the `LookDecision`:
	- If there’s a player in sight, it will go to `ChaseScanner` state (chase and fire).
	- If there's no player in sight, `RemainInState`: `PatrolScanner` state and repeat. 
4. If it arrives at `ChaseScanner` State, it will perform `AttackAction` similar to the Chaser Tank and then perform another `LookDecision`:
	- If player’s no longer within sight then it will go to an `AlertScanner` state where it will perform **two** Decisions — one after another:
	    - `ScanDecision` (rotate in place to search for player) and then, 
	    - `LookDecision`. 
	    - In `ScanDecision`, there’s a *timeout* involved. 
	    > Find which script keeps track of the time elapsed while Scanning? 
	- Else, if Player is still spotted within range then `RemainInState`: `ChaseScanner`. 
	- The tank will go either to back to `PatrolScanner`, remain in `ScanScanner` state, or go `ChaseScanner` State accordingly depending on the outcome of these two decisions. 

The green state "Patrol" above corresponds to the Scriptable Object `PatrolScanner`:
<img src="https://www.dropbox.com/s/ck8c6zkz8s6huqv/9.png?raw=1"  class="center_ninety"/>

The red state "Chase" in the diagram above corresponds to the Scriptable Object `ChaseScanner`:
<img src="https://www.dropbox.com/s/kjzv57cd3j8cn8i/10.png?raw=1"  class="center_ninety"/>

The yellow state "Alert" in the diagram above corresponds to the Scriptable Object `AlertScanner` in the Project:

<img src="https://www.dropbox.com/s/h3mutzffgaglxmo/11.png?raw=1"  class="center_ninety"/>

Hence the StateController component in AITankScanner is set to start at `PatrolScanner` as it's **starting state**:
<img src="https://www.dropbox.com/s/a1xnl2w0eh67wmp/12.png?raw=1"  class="center_ninety"/>


# Further Details
This section explains further on the details between state transitions: `Decision` that results in `RemainInState` or some new State. 


## TransitionToState in StateController.cs
In `StateController.cs` script, we have this method:

```java
   public void TransitionToState(State nextState)
   {
       if (nextState == remainState) return;
       currentState = nextState;
       if(currentState.name == "ChaseScanner" || currentState.name == "ChaseChaser")gameObject.GetComponent<Animation>().Play();
       OnExitState();
   }
```

It is clear that if `nextState` equals to `remainState` (which is a **public** variable with `RemainInState` Scriptable Object Linked Up in the Inspector) as shown for AITankScanner as example:

<img src="https://www.dropbox.com/s/a1xnl2w0eh67wmp/12.png?raw=1"  class="center_ninety"/>


..then we don’t need to execute the rest of the instructions anymore. 

But why can't we simply put the **same** state, for example `PatrolScanner` as the `FalseState` instead? Why do we have to use `RemainInState`?
<img src="https://www.dropbox.com/s/ck8c6zkz8s6huqv/9.png?raw=1"  class="center_ninety"/>


The difference between using `RemainInState` and just setting the `False State` as itself (e.g: `PatrolScanner` for above's example) depends on whether `OnExitState()` is called in the last instruction of `TransitionToState` in `StateController.cs`. 

This design of **not calling** `OnExitState` when `RemainInState` is handy if we want to keep track of some kind of **countdown** (`timeElapsedInState`) while being in a State for more than a frame, or if we need to clean up some variables while remaining in the State for more than one Update frame. In `StateController.cs`, `OnExitState` is used to cleanup the timer to keep track how long exactly we have been in the `AlertScanner` state:
```java
	void OnExitState()
	{
		stateTimeElapsed = 0;
	}
```

