---
title: Object Pooling
permalink: /unity/unity_for_children_2
key: unity-unity_for_children_2
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

Object pooling is a design pattern that you can use to Object Pooling to optimize your projects by lowering the burden that is placed on the CPU when having to rapidly **instantiate** and **destroy** GameObjects. It is particularly useful for top-down bullet-spray games, or games that have swarms of monsters that are constantly created and destroyed at runtime. 

Although Super Mario Bros do not have anything that needs to be spawned and destroyed many times (think hundreds!)  at runtime, we can try to apply this concept to manage various enemies in the game. 

The main idea of Object Pooling is as follows:
* Instantiate `N` objects at `Start()`, but render them **inactive**. Place all of them in a *pool*.
* Activate each objects at runtime accordingly, instead of instantiating new ones. This action removes *available* objects from the *pool*. 
* After we are done with these objects, deactivate them. This essentially returns the object back to the *pool*, ready to be reused next time. 
* The *pool* may run out of objects to be activated eventually, and we can expand the pool at runtime. This requires instantiation of new gameObjects obviously, so try to reduce the need to do so and instantiate enough relevant game objects at `Start()`. 

Create a **new** script called ObjectPooler.cs. 

## C#: Enumeration Types
We begin by stating the types of objects that we can instantiate in our Pool. We can skip this step and use Tags instead, but it might be more efficient to use `enum` type.

> An _enumeration type_ (or _enum type_) is a [value type](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types) defined by a set of named constants of the underlying [integral numeric](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types) type.

As an example, we have two enemy types: gomba and green. You can define your own integral numeric type on the right hand side,  but for sanity sake let's use something intuitive as follows: 
```java
public  enum ObjectType{
	gombaEnemy =  0,
	greenEnemy =  1
}
```
Paste the above code outside `ObjectPooler` class in `ObjectPooler.cs` script. 

## C#: Helper Class
We can define other classes as well, without having to inherit MonoBehavior.  These classes are our helper class. We need to create two of these:
* A class to define the data structure of an Object *metadata* to be spawned into the pool
* A class to define the data structure of an Object in the pool

For the former, we can define something like this:
```java
[System.Serializable]
public  class ObjectPoolItem
{
	public  int amount;
	public  GameObject prefab;
	public  bool expandPool;
	public  ObjectType type;
}
```

The attribute `[System.Serializable]` indicates that a class or a struct can be serialized. In laymen terms: **visible** and **customisable** in the **inspector** when declared as a public instance `public ObjectPoolItem i` later on. 

For the latter, we have the following:
```java
public  class ExistingPoolItem
{
	public  GameObject gameObject;
	public  ObjectType type;

	// constructor
	public  ExistingPoolItem(GameObject gameObject, ObjectType type){
		// reference input
		this.gameObject  =  gameObject;
		this.type  =  type;
	}
}
```

## C#: List
Under ObjectPooler class, we declare the following variables to hold different kinds of object *metadata* in the pool and references to objects spawned in the pool:

```java
public  List<ObjectPoolItem> itemsToPool; // types of different object to pool
public  List<ExistingPoolItem> pooledObjects; // a list of all objects in the pool, of all types
```

So for example, if we have **two** types of monsters: Gomba and Green, the number of elements in `itemsToPool` is exactly **two** `ObjectPoolItems`, each containing the metadata for each monster in the form of `ObjectPoolItem`. 

Now suppose we want to have at maximum **three** Gombas and **six** Green spawned (but deactivated at first), then the number of elements in pooledObjects is **nine**, each in the form of `ExistingPoolItem`. 


We then can implement the `Awake()` method of ObjectPooler class, where we spawn all items for each `ObjectPoolItem` and put them inside `pooledObjects` List. 
```java
void  Awake()
{
	pooledObjects  =  new  List<ExistingPoolItem>();
	foreach (ObjectPoolItem item in  itemsToPool)
	{
		for (int i =  0; i  <  item.amount; i++)
		{
			// this 'pickup' a local variable, but Unity will not remove it since it exists in the scene
			GameObject pickup = (GameObject)Instantiate(item.prefab);
			pickup.SetActive(false);
			pickup.transform.parent  =  this.transform;
			ExistingPoolItem e =  new  ExistingPoolItem(pickup, item.type);
			pooledObjects.Add(e);
		}
	}
}
```

Notice that we created a **local** variable `e`, that contains a **reference** to a newly instantiated `ExistingPoolItem` object *inside this for-loop*, and then we add that reference to `pooledObjects` list. In good old C, you will end up with the horrible `segmentation fault` because this reference will be *out of scope*. C# variables are also generally **scoped** within the nearest set of `{}`s. However since we are adding it to the List, C# is **smart** **enough** to know that you still need the content of `e` later on using `pooledObjects[i]` and therefore allocates the memory space differently. `e` does not exist anymore, i.e: you can't simply `Debug.Log(e)` after the loop exists, but its content stays in `pooledObjects`.  It is out of our syllabus, but we just want you to take a moment to *appreciate* how this feature makes our lives so much easier. 

Anyway, you can have a neater code with inline instantiation instead:

```java
pooledObjects.Add(new  ExistingPoolItem(pickup, item.type));
```

## Getting the Pooled Object
Create a public method to return GameObject reference of one of the requested objects in the pool when available:

```java
public  GameObject  GetPooledObject(ObjectType type)
{
	// return inactive pooled object if it matches the type
	for (int i =  0; i  <  pooledObjects.Count; i++)
	{
		if (!pooledObjects[i].gameObject.activeInHierarchy  &&  pooledObjects[i].type  ==  type)
		{
			return  pooledObjects[i].gameObject;
		}
	}
	return null;
}
```

There's no method `returnPooledObject` because the *user* of the pooled object should render the object back as inactive. Deactivating the pooled objects essentially "returns" the object back to the pool, which makes it convenient to use. 

> Any script controlling the pooled object must be aware of this setup, such as resetting all its state and `transform.position` when deactivated so that it can be reused another time. 

We can also expand the Pool if we didn't instantiate enough objects in the Pool at `awake`. Notice that `ObjectPoolItem` as the attribute `bool expandPool`. We can add the following check after the `for` loop but before `return null` to instantiate the requested object at runtime if `expandPool == true`. 

```java
// this will be called when no more active object is present, item to expand pool if required
foreach (ObjectPoolItem item in itemsToPool)
{
	if (item.type == type)
	{
		if (item.expandPool)
		{
			GameObject pickup = (GameObject)Instantiate(item.prefab);
			pickup.SetActive(false);
			pickup.transform.parent  =  this.transform;
			pooledObjects.Add(new  ExistingPoolItem(pickup, item.type));
			return  pickup;
		}
	}
}
```

The drawback with this method is that you have to loop through both `pooledObjects` and `itemsToPool` whenever this method is called *and* there's no available object to return. You can use better data structures to avoid this, but nevertheless it shouldn't be a problem when run on modern computers. 

## C#: Static Variable
Finally, we want to be able to access the ObjectPooler instance fast and since there should only be one instance of this per Scene, we can create a static variable in ObjectPooler.cs too and set it in `Awake`:

```java
public static ObjectPooler SharedInstance

void Awake(){
	SharedInstance = this;
	// other instructions 
	....	
}
```

## Calling GetPooledObject()
To demonstrate how we can utilise the object pooler, create a new script `SpawnManager.cs`. This script will call `GetPooledObject()` method in `ObjectPooler.cs` (obtain objects from the pool). Add the following method in `SpawnManager.cs`:

```java
void  spawnFromPooler(ObjectType i){
	// static method access
	GameObject item =  ObjectPooler.SharedInstance.GetPooledObject(i);
	if (item  !=  null){
		//set position, and other necessary states
		item.transform.position  =  new  Vector3(Random.Range(-4.5f, 4.5f), item.transform.position.y, 0);
		item.SetActive(true);
	}
	else{
		Debug.Log("not enough items in the pool.");
	}
}
```
> Notice that we do not need to get the instance reference for `ObjectPooler` beforehand since we utilise the `static ObjectPooler` reference we created earlier. 

Then we can simply test spawning something in `Awake()`:

```java
// spawn two gombaEnemy
for (int j =  0; j  <  2; j++)
	spawnFromPooler(ObjectType.gombaEnemy);
```

## Using ObjectPooler.cs
Before we can use `ObjectPooler.cs` as a component, we need to prepare the prefabs for each object type: `gombaEnemy` and `greenEnemy`. Create these two prefabs with any sprite you want. Here's our sample for `gombaEnemy`:

<img src="https://www.dropbox.com/s/6n0zxn7c0vayiy8/6.png?raw=1"  class="center_ninety"/>

and our sample for `greenEnemy`:

<img src="https://www.dropbox.com/s/sgt9b13oy9fj6xg/7a.png?raw=1"  class="center_ninety"/>

We added Trigger Collider and Kinematic Rigidbody components to each of the prefab because we want to detect collision with the player later on in the script, etc. You can add any other components that you want depending on how you want to control these enemies. 

Then create an Empty game object in the scene, name it `EnemySpawnPool` and attach `ObjectPooler.cs` script on it. Set up the serializable `itemsToPool` variable accordingly:

<img src="https://www.dropbox.com/s/gzcfmz12o5ikgky/8.png?raw=1"  class="center_ninety"/>

 > If you have more enemyTypes, feel free to implement those as you deem fit. 

Finally, create another Empty GameObject in the scene called `EnemySpawnManager` as shown in the screenshot above, and attach the `SpawnManager.cs` script as its component. 

Test run and you shall have two cute `gombaEnemy` spawned in your scene (or whichever `enemy type` you decided to test):

<img src="https://www.dropbox.com/s/kx3q40fszmt0hzk/9.png?raw=1"  class="center_ninety"/>

Under `EnemySpawnPool`, you can also observe other inactive game objects that are available in your *pool*. You can test further by spawning more enemies at runtime or test the `expandPool` feature. With that, we conclude this section about Object Pooling. 
