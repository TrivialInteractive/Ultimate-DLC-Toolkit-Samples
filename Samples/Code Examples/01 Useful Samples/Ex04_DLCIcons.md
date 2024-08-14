This example shows how you can access the icons that you assign to the DLC Profile asset at runtime in order to display them as part of the UI or similar. Firstly you will need access to the `IDLCIconProvider` which can be retrieved from the `DLCContent`, and for this example we assume that the DLC content has already been loaded as per previous examples:

```cs
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
        void Start()
        {
                // This example assumes that the DLC has already been loaded
                DLCContent dlc = ...

                // Get a reference to the icon provider
                IDLCIconProvider icons = dlc.IconProvider;
        }
}
```

Next we can check if the DLC contains any icons for the built in icon names (Small, Medium, Large and ExtraLarge)

```cs
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
        void Start()
        {
                // This example assumes that icons has been assigned as in the above example
                IDLCIconProvider icons = ...

                // Check if any icons exist
                Debug.Log("Has Small = " + icons.HasIcon(DLCIconType.Small));
                Debug.Log("Has Medium = " + icons.HasIcon(DLCIconType.Medium));
                Debug.Log("Has Large = " + icons.HasIcon(DLCIconType.Large));
                Debug.Log("Has Extra Large = " + icons.HasIcon(DLCIconType.ExtraLarge));
        }
}
```

Similarly we can also load the icons:

```cs
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
        void Start()
        {
                // This example assumes that icons has been assigned as in the above example
                IDLCIconProvider icons = ...

                // Check if any icons exist
                Texture2D small = icons.LoadIcon(DLCIconType.Small);
                Texture2D medium = icons.LoadIcon(DLCIconType.Medium);
                Texture2D large = icons.LoadIcon(DLCIconType.Large);
                Texture2D extraLarge = icons.LoadIcon(DLCIconType.ExtraLarge);
        }
}
```

There is also the possibility of async loading to prevent blocking the main thread as much. Note that the createion and upload to GPU of the texture has to be handled on the main thread, so there may be a small delay on the main thread while that completes, but the loading is shifted to a background thread:

```cs
using System.Collections
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
        IEnumerator Start()
        {
                // This example assumes that icons has been assigned as in the above example
                IDLCIconProvider icons = ...

                // Check if any icons exist
                DLCAsync<Texture2D> request = icons.LoadIcon(DLCIconType.Small);

                // Wait for load to complete
                yield return request;

                // Check for loaded and get the texture
                if(request.IsSuccessful == true)
                {
                        // Get the texture
                        Texture2D small = request.Result;
                }

                // The sample priniciples apply for Medium, Large and ExtraLarge icons too, but have been omitted for conciseness
        }
}
```
