Ultimate DLC Toolkit includes built-in DRM support for a few DRM service providers such as Steam and Google Play. You can however add support for another service of your own choosing if required by simply implementing and registering the `IDRMProvider` interface. The implementation can implement the following members related to DRM:

* **GetDLCUniqueKeysAsync: (Async, Optional)** Fetches all unique keys for available DLC via the DRM provider. Many DRM services such as Steam provide a means of listing all published DLC, however a few services may not support this, and for that reason it is optional.
* **IsDLCAvailableAsync: (Async, Required)** Checks whether the specified DLC unique key is owned by the user and installed on the device.
* **GetDLCStream: (Async, Required)** Get the DLC IO stream for the installed DLC from the DLC unique key provided.
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
