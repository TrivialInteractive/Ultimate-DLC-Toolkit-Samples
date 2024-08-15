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

Ultimate DLC Toolkit also supports async unloading so that the game thread is not interrupted. You can optionally wait on the unload operation also, or just fire and forget:

```cs
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	// For this example we assume that this field stores an already loaded DLC
	private DLCContent content = ...

	void OnDestroy()
	{
		StartCoroutine(DestroyAsync());
	}

	IEnumerator DestroyAsync()
	{
		// Should we also unload assets that are currently in use
		// For example if a DLC prefab instance is in use.
		bool withAssets = true;

		// Request unload
		DLCAsync async = content.UnloadAsync(withAssets);

		// We can wait for it to finish if required
		// This is optional and you can just use `content.UnloadAsync(withAssets)` if you are not interested in running code after the unload has completed
		yield return async;

		// Run code after the unload has finished
		Debug.Log("Completed unloading DLC async!");
	}
}
```
