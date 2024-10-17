In most cases it is recommended to use DRM for managing and loading DLC content, but in some cases you may need more control over finding and loading local DLC that has been shipped with the game, or installed at some point. 
For that reason we have created the `DLCDirectory` class which acts as a virtual DLC file browser to make it easier to find, list and load DLC content from a local folder.
The DLC directory is only interested in working with valid loadable DLC files. You can place any other type of file in the same directory along side your DLC, and those fles will simply be ignored by all API's, even if they have the same file extension as a valid DLC.  
First you need to create a new instance and provide the folder path where DLC content may be stored. it is also possible to search in subfolders if required as shown here:
```cs
using DLCToolkit;
using UnityEngine;
using System.IO;

public class Example : MonoBehaviour
{
        DLCDirectory directory = null;
        void Start()
        {
                // Create a directory from a relative folder path#
                // Note that the folder path must exist before hand, otherwise an exception will be thrown
                directory = new DLCDirectory("DLC");

                // Or create from a full path and search in sub folders also
                directory = new DLCDirectory(Directory.GetParent(Application.dataPath).FullName, SearchOption.AllDirectories);
        }
}
```
Once we have created a directory, we can now discover and access valid DLC content that may be located inside that folder using the available API's:
```cs
using DLCToolkit;
using UnityEngine;

public class Example : MonoBehaviour
{
        // Assumes that it has been initialized as above
        DLCDirectory directory = ...;
        void FindDLC()
        {
                // Check whether DLC with the specified name or unique key is within the directory
                bool isAvailable = directory.HasDLC("MyDLCName", null);
                isAvailable = directory.HasDLC("my_dlc_key");

                // Get the file path for the DLC with the specified name or unique key
                string dlcPath = directory.GetDLCFile("MyDLCName", null);
                dlcPath = directory.GetDLCFile("my_dlc_key");

                // Get the DLC metadata only - performs a quick partial load to acces only the metadat portion of the DLC, useful for display info about the DLC before loading it
                IDLCMetadata dlcMetadata = directory.GetDLC("MyDLCName", null);
                dlcMetadata = directory.GetDLC("my_dlc_key");
        }
}
```
Additionally there are API's to get or enumerate all DLC content that is found in the directory:
```cs
using DLCToolkit;
using UnityEngine;

public class Example : MonoBehaviour
{
        // Assumes that it has been initialized as above
        DLCDirectory directory = ...;
        void FindDLC()
        {
                Find or enumerate all DLC file paths located in the directory
                string[] dlcPaths = directory.GetDLCFiles();
                IEnumerable<string> dlcPathsEnumerated = directory.EnumerateDLCFiles();

                // Find or enumerate all DLC unique keys located in the directory
                string[] dlcUniqueKeys = directory.GetDLCUniqueKeys();
                IEnumerable<string> dlcUniqueKeysEnumerated = directory.EnumerateDLCniqeKeys();

                Finally we can get the metadata or all DLC located in the directory
                IDLCMetadata[] dlcMetadata = directory.GetAllDLC();
                IEnmerable<IDLCMetadata> dlcMetadataEnmerated = directory.EnmerateAllDLC();
        }
}
```
Finally it is possible to enable events for when DLC content is installed or removed in the DLC directory. This can be useful if you want to support hot swapping DLC for quick testing, or if you want to support install on demand, or any similar use case.
First you will need to enable the directory events feature via the constrctor:
```cs
using DLCToolkit;
using UnityEngine;

public class Example : MonoBehaviour
{
        DLCDirectory directory = null;
        void Start()
        {
                // Make sure we enable the raise events option in order to receive events when the DLC directory files change
                bool raiseEvents = true;
                directory = new DLCDirectory("My/Path", SearchOption.TopDirectoryOnly, null, raiseEvents);
        }
}
```
Now we can subscribe to the events which will be fired when the file contents of the DLC directory have changed. Note that these events will only fire for valid DLC format files:
```cs
using DLCToolkit;
using UnityEngine;

public class Example : MonoBehaviour
{
        DLCDirectory directory = null;
        void Start()
        {
                // Assumes that it has been initialized as above
                directory = ...

                // Add listeners via Unity events
                directory.OnDLCFileAdded.AddListener(OnDLCAdded);
                directory.OnDLCFileDeleted.AddListener(OnDLCRemoved);
        }

        void OnDLCAdded(string path)
        {
                Debug.Log("DLC was added: " + path);
        }

        void OnDLCRemoved(string path)
        {
                Debug.Log("DLC was removed: " + path);
        }
}
```
