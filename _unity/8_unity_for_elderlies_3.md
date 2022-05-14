---
title: Skipping Remaning Transitions
permalink: /unity/unity_for_elderlies_3
key: unity-unity_for_elderlies_3
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


## Skipping Remaining Transitions 

In `State.cs`, we have the following check transition method where we check the `bool transitionStateChanged` inside `controller`, due to the effect of calling `TransitionToState` at the end of the `for`-loop. 

```java
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

This instruction: `if (controller.transitionStateChanged)` is used to **skip** the need for going through other `Decisions` once a previous `Decision` leads to a **NEW** next state. Otherwise if the previous Decisions always ends up with `RemainInState`, then we continue iterating through other Decisions to check if any of them leads to a **new next state**.



# Summary
There's no checkoff with this lab. In a nutshell, we have learned how to use ScriptableObject to create a simple Finite State Machine:
1.  We can create various types of Abstract classes inheriting ScriptableObject: Action, Decision, and State
2. Create new scripts inheriting either of the above for **specific** usage
3.  How to use them along with a generic `StateController` specific to the tank
4.  How to trace back the usage of Action, Decision, and State Scriptable object instances during each `Update` call of `StateController`


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5NjQ2NjY4MTIsNjcyMTA5OTU0LC0yMD
M4MTg1MTA5XX0=
-->