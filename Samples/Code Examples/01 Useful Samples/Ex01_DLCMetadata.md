This short example shows how you can access the built in metadata that Ultimate DLC Toolkit includes. Additionally you can included custom metadata in DLC if required as shown in the next code example. 
This example assumes that you have followed one of the previous load DLC samples in order to load `DLCContent` into the game

```cs
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
        void Start()
        {
                // This example assumes that the DLC has already been loaded
                DLCContent dlc = ...

                // Now we can access the metadata for the DLC
                IDLCMetadata metadata = dlc.Metadata;


                // Get the display name of the DLC
                Debug.Log("Name: " + metadata.NameInfo.Name);

                // Get the unique key of the DLC
                Debug.Log("Unique Key: " + metadata.NameInfo.UniqueKey);

                // Get the version of the DLC
                Debug.Log("Version: " + metadata.NameInfo.Version.ToString());

                // Get the guid of the DLC
                Debug.Log("Guid: " + metadata.Guid);

                // Get the description of the DLC
                Debug.Log("Description: " + metadata.Description);

                // Get the developer of the DLC
                Debug.Log("Developer: " + metadata.Developer);

                // Get the publisher of the DLC
                Debug.Log("Publisher: " + metadata.Publisher);

                // Get the Ultimate DLC Toolkit version used to build this DLC
                Debug.Log("Toolkit Version: " + metadata.ToolkitVersion.ToString());

                // Get the Unity version used to build this DLC - Should usually be the same version as your base game
                Debug.Log("Unity Version: " + metadata.UnityVersion);

                // Get the content flags describing the included content types
                Debug.Log("Content Flags: " + metadata.ContentFlags);

                // Get the time stamp for when the DLC was built
                Debug.Log("Build Time: " + metadata.BuildTime);

                // Check whether the DLC content was distributed as part of the base game or whether it was installed externally by a DRM service for example
                Debug.Log("Shipped With Game: " + metadata.ShippedWithGame);

                // Get the network unique identifier that can be used accross network clients to determine if the same DLC is in use. Build time stamp should be true if you need to match the exact DLC build accross clients
                bool includeBuildTimeStamp = true;
                Debug.Log("Network Unique Identifier: " + metadata.GetNetworkUniqueIdentifier(includeBuiltTimeStamp));

                // Get the network unique identifier hash. Derived from the same information above, however is guarenteed to be a fixed length string
                Debug.Log("Network Unique Identifier Hash: " + metadata.GetNetworkUniqueIdentifierHash());
        }
}
```
