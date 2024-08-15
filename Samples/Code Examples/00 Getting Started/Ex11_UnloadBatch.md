This quick example shows how you can unload multiple DLC in batches with a single call.

```cs
using System.Collections.Generic;
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	void OnDestroy()
	{
		// This example assumes that you already have an enumerable of loaded dlc's (It can be an array, list or anything really)
		IEnumerable<DLCContent> dlcs = ...

		// Should we also unload assets that are currently in use
		// For example if a DLC prefab instance is in use.
		bool withAssets = true;

		// Unload all as a single operation
		DLC.UnloadDLCBatch(dlcs, withAssets);
	}
}
```

It is also possible to batch unload asynchronously so that the game thread is not interrupted. You can optionally wait on the unload operation which will only continue once all DLCs have been unloaded, or just fire and forget:

```cs
using System.Collections;
using System.Collections.Generics;
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	void OnDestroy()
	{
		StartCoroutine(DestroyAsync())
	}

	IEnumerator DestroyAsync()
	{
		// This example assumes that you already have an enumerable of loaded dlc's (It can be an array, list or anything really)
		IEnumerable<DLCContent> dlcs = ...

		// Should we also unload assets that are currently in use
		// For example if a DLC prefab instance is in use.
		bool withAssets = true;

		// Unload all as a single operation
		DLCAsync async = DLC.UnloadDLCBatch(dlcs, withAssets);

		// We can wait for it to finish if required
		// This is optional and you can just use `DLC.UnloadDLCBatch(dlcs, withAssets)` if you are not interested in running code after the unload has completed
		yield return async;

		// Run code after the unload has finished
		Debug.Log("Completed batch unloading DLC async!");
	}
}
```
