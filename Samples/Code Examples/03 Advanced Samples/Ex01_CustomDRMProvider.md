Ultimate DLC Toolkit includes built-in DRM support for a few DRM service providers such as Steam and Google Play. You can however add support for another service of your own choosing if required by simply implementing and registering the `IDRMProvider` interface. The implementation can implement the following members related to DRM:

* **GetDLCUniqueKeysAsync: (Async, Optional)** Fetches all unique keys for available DLC via the DRM provider. Many DRM services such as Steam provide a means of listing all published DLC, however a few services may not support this, and for that reason it is optional.
* **IsDLCAvailableAsync: (Async, Required)** Checks whether the specified DLC unique key is owned by the user and installed on the device.
* **GetDLCStreamAsync: (Async, Required)** Get the DLC IO stream for the installed DLC from the DLC unique key provided.
* **RequestInstallDLCAsync: (Async, Optional)** Request that the DRM service begins installing the DLC on demand if not already installed and if the service supports it.
* **RequestUninstallDLC: (Optional)** Request that the DRM service begins uninstalling the DLC from the user device if supported.
* **TrackDLCUsage: (Optional)** Inform the DRM service when the DLC is loaded or unloaded by the game so that it is posisble to track DLC play time or similar. For example, Steam supports tracking the amount of play time for a particular DLC.

Here is an example interface implementation using a fictional example API, but actual implementation may vary depnding upon the service you are trying to integrate. Some integrations may be more difficult than others, but if you are having trouble then feel free to reach out via one of the support channels and we may be able to help:

```cs
using UnityEngine;
using DLCToolkit;
using DLCToolkit.DRM;

public class ExampleDRM : IDRMProvider
{
        // Fetch DLC unique keys from some fictional API
        // This is optional and you can throw `NotSupportedException` if you do not wish to implement
        public DLCAsync<string[]> GetDLCUniqueKeysAsync(IDLCAsyncProvider asyncProvider)
        {
                 return DLCAsync<string[]>
                        .Completed(true, SomeAPI.GetAllDLC()
                        .Select(d => d.id)
                        .ToArray());
        }

        // Check if the DLC is available
        public DLCAsync<bool> IsDLCAvailableAsync(IDLCAsyncProvider asyncProvider, string uniqueKey)
        {
                DLCAsync<bool> async = new ();

                // Launch some task that makes a request on a background thread
                SomeAPI.FetchDLCStatus(uniqueKey, (Status s) =>
                {
                        // Mark async as completed or failed - Required or the method will wait forever when called
                        if(s == Status.Owned)
                                async.Complete(true, true);
                        else
                                async.Error("DLC is not owned by the user");                        
                });
                return async;
        }

        public DLCAsync<DLCStreamProvider> GetDLCStream(IDLCAsyncProvider asyncProvider, string uniqueKey)
        {
                // Call some fictional API to find out the install location of the DLC
                // If the DRM service requires an async call to determine install location, we would recommend performing that in `GetDLCUniqueKeysAsync` and caching the results in a dictionary or similar
                string dlcPath = SomeAPI.GetDLC(uniqueKey).installPath;
                return DLCAsync<DLCStreamProvider>.Completed(true, DLCStreamProvider.FromFile(dlcPath));
        }

        public DLCAsync RequestInstallDLCAsync(IDLCAsyncProvider asyncProvider, string uniqueKey)
        {
                // Don't wish to implement this, or it is not supported by the service provider
                throw new System.NotSupportedException();
        }

        public void RequestUninstallDLC(string uniqueKey)
        {
                // Don't wish to implement this, or it is not supported by the service provider
                throw new System.NotSupportedException();
        }

        public void TrackDLCUsage(string uniqueKey, bool isInUse)
        {
                // Inform some fictional API that the DLC is in use
                SomeAPI.SetDLCUsage(uniqueKey, isInUse ? PlayStatus.Playing : PlayStatus.NotPlaying);
        }
}
```

Once you have created your custom DRM provider, the next step will be to register that when the game starts. Ideally it will be the first call you make to the Ultimate DLC Toolkit API, so it is recommended that you place it at the Awake or Start method of a script which runs early in your game startup.

```cs
using UnityEngine;
using DLCToolkit;
using DLCToolkit.DRM;

public class Example : MonoBehaviour
{
        void Start()
        {
                // Register the custom DRM provider with DLC toolkit
                DLC.RegisterDRMProvider(new ExampleDRM());

                // Note: you can register a different DRM provider at any time if required using the same approach
        }
}
```

Alternativley if you are creating DRM providers for a few different build targets (If you are supporting both desktop and mobile for example), then you may prefer to use a `IDRMServiceProvider` in addition, where you can determine the DRM to use based on preprocessor directives, or on runtime platform conditions if you prefer:

```cs
using UnityEngine;
using DLCToolkit;
using DLCToolkit.DRM;

public class ExampleDRMDesktop : IDRMProvider { /* Implementation omitted */ }
public class ExampleDRMMobile : IDRMProvider { /* Implementation omitted */ }

public class ExampleDRMService : IDRMServiceProvider
{
        // Will only be called the first time the unified DRM API is used
        public IDRMProvider GetDRMProvider()
        {
#if UNITY_STANDALONE
                return new ExampleDRMDesktop();
#elif UNITY_IOS || UNITY_ANDROID
                return new ExampleDRMMobile()
#else
                // No DRM available
                throw new NotSupportedException("DRM is not available on this platform");
#endif
        }
}

public class Example : MonoBehaviour
{
        void Start()
        {
                // Register the custom DRM service provider with DLC toolkit
                DLC.RegisterDRMServiceProvider(new ExampleDRMService());
        }
}
```

##### DLCAsync Tips
As per the above example code, many of the interface calls are async and use some variation of the `DLCAsync` type as the return value. There are static convenience methods such as `Completed` and `Error` as used above for quickly returning an async object with a given state.  
  
Additionally you can create an async object using the constructor as shown below where you can provide an optional bool `awaitable`. If your DRM provider uses coroutines to send requests or similar, then it is highly recommended that you disable the `awaitable` option, as otherwise it can cause the application to freeze when using a synchronous call (`LoadDLC` for example will use async methods in the DRM provider, but will attempt to block the main thread until they complete) until the operation times out (10 seconds by default). This is because DRM calls must be awaited in synchronous methods, which means blocking the main thread until completed, which in the case of the coroutine will mean that it is not updated by Unity during that time and will not be able to be completed.
```cs
// Create a non awaitable operation - will throw an exception when used in a synchronous call to avoid freeze until timeout
DLCAsync<string[]> asyncNonAwaitable = new DLCAsync<string[]>(false);

// Create an awaitable operation (Default) - If you are using C# threads or Task.Run for the API request, then it is possible for usually async calls to be awaited by blocking the main thread in the case of a synchronous call such as `LoadDLC`
DLCAsync<bool> asyncAwaitable = new DLCAsync<bool>()
```

###### Running a coroutine as part of the async API.  
  
In many cases it is ideal to use a coroutine to make the async request of the DRM provider, in the case that you are using `UnityWebRequest` or similar as an example to make an API call. In that case a common pattern is to hand off an async operation to the coroutine which will update the operation upon completion. You will note that all Async methods pass in a `IDLCAsyncProvider` for this purpose:
```cs
DLCAsync<string> ExampleCoroutineAsync(IDLCAsyncProvider asyncProvider)
{
    // Create an async object - disable waiting as it could cause the game to freeze when using coroutines
    DLCAsync<string> async = new DLCAsync<string>(false);

    // Run coroutine - asyncProvider allows coroutines to be easily started
    asyncProvider.RunAsync(ExampleRoutine(async));

    // Return the async operation now - we will update it later
    return async;
}

IEnumerator ExampleRoutine(DLCAsync<string> async)
{
    using(UnityWebRequest request = UnityWebRequest.Get("mydlcservice.com"))
    {
        // Wait for completed
        yield return request.SendWebRequest();

        // IMPORTANT - We need to call `Complete` or `Error` on the async operation once finished, otherwise it will wait forever
        async.Complete(request.result == UnityWebRequest.Result.Success, request.downloadHandler.text);
    }
}
```

###### Running a threaded call

In other cases such as using normal C# web requests, it is possible that a true async call will be made where the code runs on a separate thread, unlike a coroutine which runs on the main thread but uses a staggered update method. In such a case a common pattern for implementing the async API could be something like this:
```cs
DLCAsync<string> ExampleThreadAsync(IDLCAsyncProvider asyncProvider)
{
    // Create an async object
    DLCAsync<string> async = new DLCAsync<string>();

    // Run some threaded call
    ThreadPool.QueueUserWorkItem((object state) =>
    {
        // Run the main call that will take up the time
        string result = DoSomeOperationThatTakesSomeTime();

        // IMPORTANT - We need to call `Complete` or `Error` on the async operation once finished, otherwise it will wait forever
        async.Complete(result != null, result);
    });

    // Return the async operation now - we will update it later
    return async;
}
```
