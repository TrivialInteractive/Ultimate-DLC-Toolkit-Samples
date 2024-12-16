This short example will show how you can use the C# async/await feature to wait for any DLC async operation to complete. In the normal Unity workflow, coroutines are commonly used to wait until an async request has completed, but many users may prefer to use the C# async/await feature to interface better with game code, or simply for better code readability.
All async operations in Ultimate DLC Toolkit will return either `DLCAsync` or `DLCAsync<>` objects which are custom enumerator objects that can be yielded in a coroutine. As of version `1.1`, there is now a `.Task` property which as the name suggests will return a C# `Task` object that can be used with async/await. 

Below is a quick code example of how you might wait for an async operation to complete, and then run further code on completion:
```cs
using System.Threading.Tasks;
using UnityEngine;
using DLCToolkit;

class Example : MonoBehaviour
{
        async void Start()
        {
                // Start the load request and await the task object
                DLCContent dlc = await DLC.LoadDLCAsync("myDLCKey").Task;

                // Check for loaded
                Debug.Log("Loaded: " + (dlc != null));
        }
}
```

Here is the same code as above but broken down a bit more so it is slightly easier to see what is going on:
```cs
using System.Threading.Tasks;
using UnityEngine;
using DLCToolkit;

class Example : MonoBehaviour
{
        async void Start()
        {
                // Start the load request
                DLCAsync<DLCContent> request = DLC.LoadDLCAsync("myDLCKey");

                // Wait for the load to complete
                DLCContent dlc = await request.Task;

                // Check for loaded
                Debug.Log("Loaded: " + (dlc != null));
        }
}
```
