---
title: C# Async-Await
permalink: /unity/unity_for_adults_3
key: unity-unity_for_adults_3
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

# C#: Async and Await

We have learned about Coroutines before, which is ideal to perform asynchronous operation since we can easily `yield` execution and resume only in the next frame or after a brief period amount of time using `WaitForSeconds`. The documentation for that can be found [here](https://docs.unity3d.com/ScriptReference/WaitForSeconds.html). 

But what if you need to perform intensive computation, for example something that requires tens of thousand of clock cycles *while still keeping your game responsive*?

To test this, create a new script called `ComputationManager.cs` with the following instructions:

```java
using System.Collections;
using System.Threading;
using UnityEngine;
using System.Threading.Tasks;

public enum method
{
    useVanilla = 0,
    useCoroutine = 1,
    useAsync = 2
}

public class ComputationManager : MonoBehaviour
{
	public  method method;
    public int size;
    private bool calculationState = false;

    void Update()
    {
        if (Input.GetKeyDown("c"))
        {
            if (!calculationState)
            {
                Debug.Log("c is pressed.");
                switch (method)
                {
                    case (method.useVanilla):
                        PerformCalculations();
                        break;
                    case (method.useCoroutine):
                        StartCoroutine(PerformCalculationsCoroutine());
                        break;
                    default:
                        break;
                }
                Debug.Log("Perform calculations dispatch done");
            }
        }

        if (Input.GetKeyDown("q"))
        {
            Destroy(this.gameObject);
        }
    }
}
```

Depending on the value of `method` we set at the inspector later on, the `Update` function is going to call the respective function. Each function performs the same amount of "work", albeit in different manners. 

Here's the implementation of the vanilla method. Nothing asynchronous here, therefore depending on the value of `size`, the game will **lag** (render unresponsive) when this method is called. 
```java
    void PerformCalculations()
    {
        System.Diagnostics.Stopwatch stopwatch = new System.Diagnostics.Stopwatch();
        stopwatch.Start();
        calculationState = true;
        float[,] mapValues = new float[size, size];
        for (int x = 0; x < size; x++)
        {
            for (int y = 0; y < size; y++)
            {
                mapValues[x, y] = Mathf.PerlinNoise(x * 0.01f, y * 0.01f);
            }
        }
        calculationState = false;
        stopwatch.Stop();
        UnityEngine.Debug.Log("Time taken: " + (stopwatch.Elapsed));
        stopwatch.Reset();
    }
```

Here's the same implementation using Coroutine:
```java

    IEnumerator PerformCalculationsCoroutine()
    {
        System.Diagnostics.Stopwatch stopwatch = new System.Diagnostics.Stopwatch();
        stopwatch.Start();

        calculationState = true;
        float[,] mapValues = new float[size, size];
        for (int x = 0; x < size; x++)
        {
            for (int y = 0; y < size; y++)
            {
                mapValues[x, y] = Mathf.PerlinNoise(x * 0.01f, y * 0.01f);
                yield return null; // takes super long, only called at 60 times a second
            }
        }
        calculationState = false;
        stopwatch.Stop();
        UnityEngine.Debug.Log("Time taken: " + (stopwatch.Elapsed));
        stopwatch.Reset();
        yield return null;
    }
```

Now attach this to the player (the player in the same scene you use for the combo above), and set `size = 1000`. Set Method to be `useVanilla`, and press the key 'c'. Observe in the console the time taken to execute the computation:

<img src="https://www.dropbox.com/s/oto51k0w1ic5op0/5.png?raw=1"  class="center_ninety"/>

Now change the method to `useCoroutine` , notice the output in the console doesn't include the time taken for the function to complete:
<img src="https://www.dropbox.com/s/t1vwzc6ljzr0npy/6.png?raw=1"  class="center_ninety"/>

In fact, you have to wait for:
$$ 
\frac{1000\times1000}{60} = 16667\text{ } seconds
$$
..because the coroutine `yield` at each inner loop (means the next loop is only resumed at the next frame). This is **way too long** of a wait even though the system stays responsive in the meantime.  


We can place `yield return null` in the outer loop as such,

```java
        for (int x = 0; x < size; x++)
        {
            for (int y = 0; y < size; y++)
            {
                mapValues[x, y] = Mathf.PerlinNoise(x * 0.01f, y * 0.01f);
            }
            yield return null; // takes super long, only called at 60 times a second
        }
```
But even this needs 16.7 seconds to complete although the system will stay responsive. 

A bigger problem: what if now `size` is set to `10000` with `useVanilla` method? 
<img src="https://www.dropbox.com/s/wuppi4wkhe8pvl1/7.png?raw=1"  class="center_ninety"/>

In our 2019 16" MBP (8 core 2.3GHz), it takes 3.9 seconds to perform this computation. During this period, the game is **unresponsive**, and this is certainly **not tolerable**. If we were to use coroutine, well, we must wait for  approximately 19 days for it to complete if `yield return null` is placed in the inner loop, and 167 seconds if `yield return null` is placed in the outer loop. 

With Coroutine, **we cannot utilise the power of our CPU** while staying responsive, and without Coroutine our game wont even stay responsive because all resources is dedicated to execute this hefty computation until completion.

In order to **utilise our CPU** while staying responsive, we can use `async` function and `await` for `Task` completion. Remember that `Update` is only called 60 times a second (where we check for inputs, execute basic game logic, etc), so there's plenty of leftover time that we can use to complete this `calculation` function. 

We can declare an `async` function with the `async` keyword:
```java
    async void PerformCalculationsAsync()
    {
    }
``` 

Calling an async function **without any await** results in synchronous execution. We need to `await` some `Task` as such:
```java
    async void PerformCalculationsAsync()
    {
        System.Diagnostics.Stopwatch stopwatch = new System.Diagnostics.Stopwatch();
        stopwatch.Start();
        var result = await Task.Run(() =>
        {
            calculationState = true;
            float[,] mapValues = new float[size, size];
            for (int x = 0; x < size; x++)
            {
                for (int y = 0; y < size; y++)
                {
                    mapValues[x, y] = Mathf.PerlinNoise(x * 0.01f, y * 0.01f);
                }
            }
            return mapValues;
        });

        calculationState = false;
        Debug.Log("Size of 2D array: " + result.GetLength(0) + " " + result.GetLength(1));
        Debug.Log("Map values first row: " + result[0, 0]);
        stopwatch.Stop();
        UnityEngine.Debug.Log("Time taken: " + (stopwatch.Elapsed));
        stopwatch.Reset();
       
    }
``` 

Now simply add another case in `Update` to test this function:
```java
		case (method.useAsync):
			PerformCalculationsAsync();
			break;
```

Testing the `async` method with `size=10000` results in the following output:
<img src="https://www.dropbox.com/s/fewqt5aqkiw5nob/8.png?raw=1"  class="center_ninety"/>

This shows that using `async` function **does not** (necessarily) make any computation time faster than the `vanilla` method, but in the meantime, the system is still **responsive**. 

> In short: the  `async`  function, very similar to JavaScript, is not executed in another thread. Instead, the function executes on the main thread, and only when the  `await`  keyword appears, the function executed may or  **MAY NOT**  on the main thread.

## Comparisons with Coroutine
This section briefly covers the comparison between the two. There's no *better or worse solution*, and you can simply choose the solution that suits your project best. The content for this section is distilled from [this](https://www.youtube.com/watch?v=7eKi6NKri6I) video. 

### Async Functions Always Complete
Async functions **always** runs into completion, while Coroutines are run on the GameObject. Therefore, **disabling** the gameobject will cause any coroutine running on it to **stop** but *doesn't exit naturally*. This can potentially result in memory leak.

For example, suppose we instantiate `RenderTexture` in a Couroutine:
```java
IEnumerator RenderEffect(RawImage r){
	var texture = new RenderTexture(1024, 1024, 0);
	try{
		for (int i = 0; i<1000; i++){
			// do something with r and texture
			yield return null;
		}
	}
	finally{
		texture.Release();
	}
}
```
As per Unity Official Documentation, `RenderTexture` is **not automatically managed**, therefore it is important to call `Release()` after we are done with it.
> As with other "native engine object" types, it is important to pay attention to the lifetime of any render textures and `release` them when you are finished using them, as they will not be garbage collected like normal managed types.

Now if the MonoBehavior (the gameobject) running this script is disabled, the `finally` clause never called, thus resulting in **memory leak**.

On the other hand, `async` function continues to run **even after the MonoBehavior is destroyed**. You can test this very easily by setting `Size = 10000`, and method: `useAsync`, run the the project quickly and press `c` then quickly stop it. The output will still appear at Console even after the program exits. This shows that `async` function **always exits**. 

<img src="https://www.dropbox.com/s/e8t0ruh810a49te/asynccomplete.gif?raw=1"  class="center_ninety"/>
 
### Cancelling Async Functions
To be more sure that Coroutines always exit, we need to be mindful to`StopCoroutine(...)` during `onDisable` the gameobject. Likewise, we can also *cancel* async functions using *cancellation tokens*.

You simply just need to declare it beforehand,
```java
CancellationTokenSource token;
```

and initialise it at `Start()`:
```java
token  =  new  CancellationTokenSource();
```

Then simply pass this token when defining `Task`, and check it wherever you want *inside the Task*,
```java
        var result = await Task.Run(() =>
        {
	        calculationState = true;
            float[,] mapValues = new float[size, size];
            // ... implementation
            for (.....){
	            // ... implementation
	            // periodically check for cancellation token request
                if (token.IsCancellationRequested)
                {
                   Debug.Log("Task Stop Requested");
                   return mapValues;
                 }
            }
            return result;
        }, token.Token);
```

... or inside the `async` function that awaits that task:
```java
        if (token.IsCancellationRequested)
        {
            Debug.Log("Task Stopped");
            return;
        }
```

We can create cancellation request as follows, for example:
```java
    private void OnDisable()
    {
        Debug.Log("itemDisabled");
        token.Cancel();
    }
```

This way, the async function will stop once the gameObject is disabled:

<img src="https://www.dropbox.com/s/20nghav88899tu0/stoptask.gif?raw=1"  class="center_ninety"/>

### Return Values in Async Functions
We cannot return anything in a Coroutine, but async functions can the following return types:
-   [`Task`](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task), for an async method that performs an operation but returns no value.
-   [`Task<TResult>`](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task-1), for an async method that returns a value.
-   `void`, for an event handler.

For example, the following async function returns a `sprite`:
```java
    async Task<Sprite> LoadAsSprite_Task(string path)
    {
        // getting sprite inside Assets/Resources/ folder
        var resource = await Resources.LoadAsync<Sprite>(path);
        return (resource as Sprite);
    }
```

We can call them as such in `Start()` (notice how the `Start` method has to be `async` now to `await` this `Task`), and print some quick test in `Update()` to **confirm** if `Update()` is run at least for one frame before `Start()` is continued, and we can obtain some information about the **return value** of `LoadAsSprite_Task` *async* function:

```java
    private bool testTask = false;
    private int frame = 0;
    async void Start()
    {
        Debug.Log("Start method begins...");
        token = new CancellationTokenSource();
        var sprite = await LoadAsSprite_Task("mushroom1");
        Debug.Log("The sprite: " + ((Sprite)sprite).name + " has been loaded.");
        Debug.Log("Start method completes in frame: " + frame.ToString());
        testTask = true;
    }
    void Update()
    {
        frame++;
        if (!testTask)
            Debug.Log("Update called at frame: " + frame);
     }
```

Here's the console output:
<img src="https://www.dropbox.com/s/qtwp708epork3bh/9.png?raw=1"  class="center_ninety"/>

It shows that `Start` is called first as usual, but **asynchronously**, allowing `Update` to advance and increase the frame value. When the sprite has been loaded, the `Start` method resumes and print the `Start method completes...` message. 

## UniTask
Finally before we conclude, we'd like to introduce you to an alternative called `UniTask`. Not only a nicer replacement for Unity Coroutine and C# async-await, UniTask also provides a nicer background *thread management*. The complete documentation and installation details can be obtained [here](https://github.com/Cysharp/UniTask). 

You can [download it as UnityAsset](https://github.com/Cysharp/UniTask/releases) and import it to your project. We won't be going into details on how to utilise UniTask in your project, only to quickly introduce to you because it is a good and popular alternative. Among other things, UniTask is capable of:
- Making all Unity AsyncOperations and Coroutines **awaitable**
- Running completely on Unity's PlayerLoop so doesn't use threads and runs on WebGL, wasm, etc
- Summoning TaskTracker window to prevent memory leaks

After importing the asset, you can declare the namespace as such:
```java
using Cysharp.Threading.Tasks;
```

You can implement an async function as usual that will return `UniTask` this time round:
```java
    async UniTask<Sprite> LoadAsSprite(string path)
    {
        // getting sprite inside Assets/Resources/ folder
        var resource = await Resources.LoadAsync<Sprite>(path);
        return (resource as Sprite);
    }
```

Then `await` that in the caller:
```java
   private async void TestUniTask()
   {	
	   // parallel load, and will complete when all of the supplied tasks have completed.
        var (a, b) = await UniTask.WhenAll(
            LoadAsSprite("goomba1"),
            LoadAsSprite("goomba2"));
        Debug.Log("The sprite: " + ((Sprite)a).name + " has been loaded.");
        Debug.Log("The sprite: " + ((Sprite)b).name + " has been loaded.");
        await  UniTask.Delay(2000); // introduce delay purposely for learning purposes
        Debug.Log("TestUniTask completed at frame: "  +  frame);
    }
```

You can simply test this in `Update()` using some flag `bool testUniTask=false` instantiated in the beginning, and then call:
```java
		frame++; 
        if (Input.GetKeyDown("t") && !testUniTask)
        {
	        Debug.Log("TestUniTask called at frame: "  +  frame);
            TestUniTask();
            testUniTask = true;
        }
```

You should see the log message in this exact sequence:
- `TestUniTask called...` 
- `The Sprite ... has been loaded`
- `The Sprite ... has been loaded`
- `TestUniTask completed...` but `frame` should've advanced by a few values. 

### Switching Between Thread Pool and Main Thread
Another cool feature of UniTask is that you can switch the current context execution to the thread pool instead of the main thread easily. For example, try out this function:

```java
    private async void TestUniTask()
    {
        Debug.Log("Frame: " + frame.ToString() + ". Task delay 2 seconds");
        await UniTask.Delay(2000);
        Debug.Log("Frame: " + frame.ToString() + ". Task delay 2 finished");
        Debug.Log("Frame: " + frame.ToString() + ". Thread sleep 2 seconds");
        await UniTask.SwitchToThreadPool();
        Debug.Log("Frame: " + frame.ToString() + ". Going to sleep");
        Thread.Sleep(2000);
        await UniTask.SwitchToMainThread();
        Debug.Log("Frame: " + frame.ToString() + ". Thread sleep done");
	}
```

And just call it at will in `Update` to test:
```java
		frame++;
        if (Input.GetKeyDown("t") && !testUniTask)
        {
            Debug.Log("TestUniTask called at frame: " + frame);
            TestUniTask();
            testUniTask = true;
        }
```

Here's a sample output:
<img src="https://www.dropbox.com/s/4kumg2fd2rxfvdo/10.png?raw=1"  class="center_ninety"/>

Observe that the `Frame` **increases** at each even though we call `Thread.Sleep(2000)` because **we have switched to thread pool** before calling that instruction. Otherwise, `Thread.Sleep(2000)` will **block** on the **main thread** instead (because we aren't `await`-ing anything in that line) and cause the  main thread to block, rendering the game unresponsive for two seconds.

# Summary 
There's no checkoff associated with this tutorial, but the contents of this tutorial will be tested for our midterms. 

# Next
In the next tutorial, we will learn about basics in 3D Unity Projects, and also learn about *pathfinding* in that environment.




<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxNDAxNDczNzcsLTkxMzkzNzIxNywtMT
UwNjU1Njc5M119
-->