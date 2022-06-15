---
title: Unity for Children
permalink: /unity/unity_for_children
key: unity-unity_for_children
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

**Learning Objectives:** Game Management and C# Basics
-   **Managing Audio**: Introduction to AudioMixer
-   **Optimising** the game: Object Pooling
-   **Managing** the game: 
	- Data containers using **scriptable objects**
	- Creating global game state and manager
	- Creating a hierarchy of managers: Central Manager, Powerup Manager, Spawn Manager, and Game Manager
- **C# Basics:** 
	- Enums
	- List
	- Switch statements
	- Static variables
	- **Interface and inheritance:** for managing multiple object types (consumables, enemies, etc), 
	- **Delegates** and **events**

# Introduction

The main purpose of this Lab is to introduce a few tools that can be used to manage the game better. For example, right now we have game states spread all over various scripts, audio source spread everywhere on each object, hard-to-read game logic, etc. We can improve the structure of the game better with the help of AudioMixer, ScriptableObject, Unity Event,  and a few other C# basics. 

If you already know most of the learning objectives of the Lab, feel free to head straight to the [Checkoff](https://natalieagus.github.io/50033/2021/01/04/unityforchildren.html#checkoff) section. As usual, you don't have to follow everything in this Lab step by step if you have your own preferences to manage your game. However it is imperative that you still know what is the content of this Lab as they are part of our course syllabus. 


# AudioMixer
The Unity AudioMixer allows you to **route** each AudioSource output in your scene, **mix**, apply **effects** on any of them and perform **mastering**. You can apply effects such as volume attenuation and pitch altering for each individual AudioSource. For example, you can reduce the background music of the game when there's a cutscene or conversation. We can mix any number of input signals from each AudioSource, and have exactly **one** output in the Scene: **the AudioListener usually attached to the MainCamera**. The figure below illustrates this scenario, taken from the [Official Unity AudioMixer Documentation](https://docs.unity3d.com/Manual/AudioMixerOverview.html):

<img src="https://docs.unity3d.com/uploads/Main/AudioMixerSignalPath.png"  class="center_ninety"/>

 We will not be going in depth about DSP concepts in Audio, and we assume that it is your job to find out which effects are more suitable for your project. However we will still be picking a few examples to illustrate how to utilize the AudioMixer. They include: lowpass and highpass, pitch shifting, reverb, and duck volume. 

Click Window >> Audio >> AudioMixer to have the AudioMixer tab open. On the left, there are four sections: **Mixers**, **Groups**, **Snapshot**, and **Views**. The first section is called Mixers, and they contain the overview of all AudioMixers in your Project. The output of each mixer is routed to a Scene's default **Audio Listener** typically attached on the **Main Camera**. 

> You can also route the output of one AudioMixer to another AudioMixer if your project is complex. Specifics can be found in the [official documentation](https://docs.unity3d.com/Manual/AudioMixerSpecifics.html). 

## AudioMixer Groups
On the third section, there's **Groups**. You can create several AudioMixer groups and form a *hierarchy*. Each group creates another bus in the Audio Group strip view, as shown:

<img src="https://www.dropbox.com/s/tcvyyieb2ezzzc2/1a.png?raw=1"  class="center_ninety"/>

The purpose of having these groups is so that the output of each AudioSource can be routed here, and effects can be applied. Effects of the parent group is applied to all the children groups. Each group has a Master group by default, which controls the attenuation (loudness) of all children Audio groups. 

**Create five audio groups as shown in the screenshot above.** You will only have Attenuation effect applied on each of them in the beginning, set at 0 dB. The attenuation is simply *relative* to one another, applied on a relative unit of measurement *decibels*. 

> **TL;DR** If you have sound group A at 0 dB, sound group B at 1 dB and sound group C at -2 dB, that means sound group B is the loudest among the three. How much louder is sound group B as compared to A? It depends on your perception. A *change* of 10 **dB** is accepted as the difference in level that is perceived by most listeners as “**twice as loud**” or “half as **loud**”

You can set the initial attenuation for each group to vary by default. For example, the volume of Background Sound by default is less (at -9 dB) than Player SFX (at -3 dB).  

## Effects
You can apply various effects within each audio group. The effects are applied from top to bottom, meaning the **order of effects** can impact the **final** output of the sound. You can drag each effect in the strip view (the view shown in the screenshot above) to reorder them. 

### Lowpass and Highpass Effect
Click on Shatter SFX and **add** the **Lowpass** effect in the Inspector. You can set two properties:
* Cutoff frequency
* Resonance 

The cutoff frequency indicates the frequency that is allowed to pass the filter, so the output will contain range of frequencies from zero up to this cutoff frequency. As for resonance, you can leave the value as 1.00 unless you're familiar with it. It dictates how much the filter’s self-resonance is **dampened**. 

> From Unity documentation: Higher lowpass resonance quality indicates a lower rate of energy loss, that is the oscillations die out more slowly.

An audio with lowpass filter effect applied will sound more dull and less sharp, for example it is ideal for effects where the game character throw blunt objects around. 

On the contrary, **Highpass** effect allows us to pass any frequency **above** the cutoff. If you'd like to pass through certain bands, let's say between 1000 Hz to 5000 Hz, then you can apply a lowpass at 5000Hz, and then a highpass at 1000Hz. The order doesn't matter in this case, as long as you set the right frequency cutoff for each type of effect. 

### Pitch Shifter
The **pitch shifter effect** allows you to change the pitch of the group **without** causing the output to sound sped up or slowed down. This is formally called as **pitch scaling**: the process of changing audio pitch without affecting its speed. It is commonly used in games to improve the game feel by making sound effects *less repetitive* and by helping to convey information to the player, for example:
* **Collision information:** the speed, material of items colliding
* **Importance** of events 
* **Response** to player inputs and actions

This *pitch shifter effect* **different** from the **pitch slider** located at the top of the group inspector. The regular pitch slider changes the pitch of the audio file by manipulating the sampling frequency of the audio output. 

> If the system's sampling rate was set at 44.1 kHz and we used a 22.05 kHz audio file, the system would read the samples *faster* than it should. As a result, the audio would sound sped up and higher-pitched. The inverse also can happen. Playing an audio file slower than it should will cause it to sound slowed down and lower-pitched. 

Add the pitch shifter effect on Background Sound group and adjust the properties accordingly:
* **Pitch**: adjust this slider to determine the pitch multiplier 
* **FFT size, Overlap, and Max Channels**: real-time pitch shifting is done using Fourier Transform. 	
	> You're not required to know the details behind pitch shifting DSP. If you're interested in the details of such implementation, you may refer to the Python implementation [here](https://lcav.gitbook.io/dsp-labs/dft/implementation). Since Unity is not open source, we do not know the exact implementation of these effects, but we can certainly learn to implement our own pitch shifter or any other audio effects by reading other research papers online. 

### SFX Reverb
This effect causes the sound group to be as if it is played within a room, having that complex echo effect. Each room sounds differently, depending on the amount of things that exist in a room, for example listening to music in the bathroom (high reverb) sounds different than listening to the same music in an open field. Routing all audio output through the same reverb filter has the effect as if each of the audio files are played in the same room. 

This is particularly handy if you have different sections in your game, e.g:  cave, open field, wooden houses, etc. For example, the player's footsteps will sound different in each scenario, despite having the same audio file for footsteps. 

**Add** a new audio group called **Reverb**, add add this SFX Reverb effect to the group. Set the properties as follows and set it as the parent group of the Background Sound. 

<img src="https://www.dropbox.com/s/pl8t30fd11kas66/2.png?raw=1"  class="center_ninety"/>

This will give us an "empty room" reverb effect for the Background Sound group since the latter is the child group of the former. 

SFX Reverb has many properties, and it is best if you find presets online to get the room effect that you want without going into the details. The [documentation](https://docs.unity3d.com/Manual/class-AudioReverbEffect.html) provides a brief description of each property, but it doesn't seem like much help for beginners who aren't familiar with the term.

### Duck Volume
Another very useful effect is duck volume, that allows the group's volume to automatically reduce when something else is playing above a certain threshold. For example, if we want to reduce the background music whenever the player is jumping. 

Add Duck Volume effect in Background Sound group, and set its threshold to be -65 dB.  
<img src="/50033/assets/images/lab4/1.png"  class="center_ninety"/>


This configuration means:
-   Any input over -65db will cause the volume to go down (**Threshold**)
-   The volume will go down by quite a high amount (**Ratio**)
-   The volume will go down very quickly (**Attack** time) at 100 ms and, after the alert has gone below -25db, go back to normal somewhat quickly (**Release** time).

Leave the other properties as is unless you know what they means. Right now we do not need it, and we are confident that you can learn them independently next time when you need it. 

> You will be faced with the warning that there's no send source connected for now. We will tackle it in this immediate next section. 

### Send Effect
Now the final thing to do is to **Send** an input to this Duck Volume effect, which is the source that can cause this audio group's volume to *duck*. Click Player SFX and add the Send effect. Select Background Sound as its Receive target and set its send level to 0.00 dB (so the background sound unit can receive the full amount of Player SFX output). 

## Routing AudioSource Output
Now go back to your scene and click on every type of AudioSource present in your scene, and set the output to each group accordingly. We can also create new ones whenever we deem appropriate. Below are the lists of AudioSources that we can have as example:
* **AudioSource** with theme.mp3 as `clip` and Background Sound group as `output`, attached to MainCamera gameobject. 
* **AudioSource** with smb_jump-small.mp3 as  `clip` and Player SFX group as `output`, attached to Mario gameobject. 
* **AudioSource** with shatter sound effect as `clip` (download it yourself) and Shatter SFX, attached to the the BreakableBrick gameobject.

You are free to create more audio mixer groups and effects, and route any AudioSource output to any group that you deem fit. 

### Checkoff Information
**As part of Checkoff,** you are required to create your own AudioMixer groups, apply 1-2 interesting effects on each group, download your own audio clips and demo it in your screen recording (make sure you record the sound as well!). 

## Snapshots
Snapshot is basically a saved state of your Audio Mixer. 

> From Unity documentation: Snapshot is a great way to define moods or themes of the mix and have those moods change as the player progresses through the game. 

For example, changing the parameters of SFX Reverb on different stages in the game, depending on the location of the player. It is more convenient than manually changing all SFX Reverb parameters one by one during runtime through scripts. 

You can change between snapshots programmatically using the following code:

```java
private AudioMixerSnapshot snapshot1; 
public AudioMixer mixer;

// instantiate somewhere in the code
snapshot1 = mixer.FindSnapshot("Snapshot1_name");

// then transition
snapshot1.TransitionTo(.5f); //transition to snapshot1
```
We already have one snapshot by default, the one with the star symbol. This is our **starting** snapshot (the state that will be used when we start the game). To create more snapshots, simply click the + button beside it and give it a good name. Highlight (click) on the new snapshot and start changing any parameters within the AudioMixer: volume, pitch, send level, wet-mix level, effect **parameters**. 
> Snapshot won't work with new effects or new group within the Mixer, only the parameters. If you create or delete groups or effects, it will be reflected on all Snapshots. 

## Views 
The last concept that can make your life easier if you have a complex project is to separate your views so you can focus on groups that matter for your current work. You can create a new View by clicking the + button, rename the view and click on it. Then start hiding some groups that you don't want to see within this view. 

For example, here we have SFX views that show only SFX related audio groups, and hide the background sound group:

<img src="https://www.dropbox.com/s/6u3e624ikapm63f/4.png?raw=1"  class="center_ninety"/>

You can also **color code** each group (right click on the group) to visually separate your audio groups. 

## More AudioMixers
You can create more AudioMixers, and route its output through existing AudioMixers (if you don't then all output of the mixers will be routed to the Scene's AudioListeners). This is useful to have better modularity and management if you have many stages in your game. For example, you can create a new AudioMixer for so-called "level 2" of your game:

<img src="https://www.dropbox.com/s/ay856vb1akjdosh/5.png?raw=1"  class="center_ninety"/>

If you need to route the output of *Level 2 Audio* to GameAudio, **drag** its entry under Mixers and **place it on GameAudio**, then select the group that you want use to receive the output of *Level 2 Audio*. This way we can **apply chain of audio effects** conveniently, with clear visual separation between each group and mixers. 

> Unless you have complex Audio system in your game, creating multiple groups and arranging their hierarchy is usually sufficient. You don't really need to route multiple mixers together. You can use different mixers for different set of AudioSources in completely different scenes instead. 