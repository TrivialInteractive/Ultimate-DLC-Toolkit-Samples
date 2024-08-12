This quick example shows how to unload DLC content from memory when it is no longer needed.
It is recommended where possible to unload any DLC content that is not currently being used by the game to keep memory usage to a minimum. 

```cs
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	// For this example we assume that this field stores an already loaded DLC
	private DLCContent content = ...

	void OnDestroy()
	{
		// Should we also unload assets that are currently in use
		// For example if a DLC prefab instance is in use.
		bool withAssets = true;

		// Request unload
		content.Unload(withAssets);
	}
}
```
