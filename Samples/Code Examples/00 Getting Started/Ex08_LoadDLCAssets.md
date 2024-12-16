This quick example shows how to load shared assets from a given DLC content.
Shared assets are non-scene and non-script assets such as prefabs, material, textures, scriptable object assets, and much more.  

Load an asset where the asset name is known ahead of time. May not be useful in most cases but could be suitable if all DLC include some predefined named asset like `index.asset` for example:

```cs
using UnityEngine;
using DLCToolkit;
using DLCToolkit.Assets;

public class Example : MonoBehaviour
{
	void Start()
	{
		// For this example we assume that the DLC content has already been loaded
		DLCContent content = ...

		// Try to find the asset with the name
		// DLCSharedAsset provides lots of useful metadata such as path, extension, type and more
		DLCSharedAsset asset = content.SharedAssets.Find("Test.png");

		// Check for found
		if(asset == null)
		{
			Debug.LogError("Asset not found: Test.png");
			return;
		}

		// Load the actual asset
		Texture2D textureAsset = asset.Load<Texture2D>();
	}
}
```

In most cases it is likely that you will not know the name of an asset ahead of type, or you may want to design your game/DLC in such a way that asset name is not important. 
To help with this case, DLC Toolkit provides a number of methods that can help find assets by a number of criteria such as assets inside a particular folder, assets of a certain type and more.
Here is a quick sample to show some of those cases which are likley to be more useful in real game scenarios:

```cs
using UnityEngine;
using DLCToolkit;
using DLCToolkit.Assets;

public class Example : MonoBehaviour
{
	void Start()
	{
		// For this example we assume that the DLC content has already been loaded
		DLCContent content = ...

		// Find all assets in the specified folder (relative to the DLC root folder)
		foreach(DLCSharedAsset asset in content.SharedAssets.FindAllInFolder("Character/PlayerSkins"))
		{
			// Check for assets of a specific type to filter further
			if(asset.AssetMainType == typeof(Material))
				Debug.Log("Player skin material: " + asset.RelativeName);
		}

		// Alternativley we could achive the same as above with this method
		foreach(DLCSharedAsset asset in content.SharedAssets.FindAllInFolderWithExtension("Character/PlayerSkins", ".mat"))
		{
			Debug.Log("Player skin material: " + asset.RelativeName);
		}

		// Only find assets of a specific type - here we use a scriptable object as the type since it is a common and recommended approach for storing new game items
		foreach(DLCSharedAsset asset in content.SharedAssets.FindAllOfType<MyScriptableItem>())
		{
			Debug.Log("New game item: " + asset.RelativeName);
		}
	}
}
```

Assets can also be loaded asynchronously which can be helpful for large assets where you don't want to freeze the game temporarily while the loading occurs. Loading is performed on a background thread and you wan wait for that to complete in a coroutine or similar:

```cs
using System.Collections;
using UnityEngine;
using DLCToolkit;
using DLCToolkit.Assets;

public class Example : MonoBehaviour
{
	IEnumerator Start()
	{
		// This example assumes that we have already found the asset to load via one of the above approaches
		DLCSharedAsset assetToLoad = ...

		// Load async will return an awaitable object
		DLCAsync<GameObject> async = assetToLoad.LoadAsync<GameObject>();

		// Wait for the load to complete - the game can continue running in the meantime
		yield return async;

		// Check for successful and get the actual asset
		if(async.IsSuccessful == true)
		{
			// Get the Unity asset
			GameObject loadedPrefab = async.Result;

			// Todo - Do something with the loaded asset
		}
	}
}
```

#### Load Asset From Any DLC (1.2 or newer)
As of version 1.2, it is now possible to load shared assets from any loaded DLC by providing the name or path of the asset. It is also possible in the case of async calls to load the DLC on demand as part of the same load call, in a similar way that addressables works.

Firstly the non-async variants only support loading assets from DLC contents that were already loaded before the call, but it can make things easier to manage since you don't have to deal with finding the correct DLCContent and DLCSharedAsset before attempting to load, as this example shows:

```cs
using UnityEngine;
using DLCToolkit;
using DLCToolkit.Assets;

public class Example : MonoBehaviour
{
	void Start()
	{
		// Load some DLC first
		DLC.Load("MyDLCKey");

		// Note that you can optionally provide a hint DLC unique key so that the load call will only search in this specific DLC content.
		// You can leave this as null to search in all loaded DLC's
		string dlcUniqueKey = null;

		// Load an asset with the following path as a game object
		// Note that the DLC containing the asset must be loaded before this call, but check out the async alternative to support on demand DLC loading
		GameObject myPrefab = DLC.LoadAsset<GameObject>("Prefabs/Truck.prefab", dlcUniqueKey);

		// Check for found
		Debug.Log("Loaded asset: " + myPrefab);
	}
}
```

Finally you can use a similar API to load the asset asynchronously. Note that the async variants differ slightly because they support loading the DLC content on demand if it is not already loaded into memory. In that regard it works in a similar way to addressables, so may be useful to users who are switching to DLC Toolkit so that they can keep a similar code base.

```cs
using System.Collections;
using UnityEngine;
using DLCToolkit;
using DLCToolkit.Assets;

public class Example : MonoBehaviour
{
	IEnumerator Start()
	{
		// Note that you can optionally provide a hint DLC unique key so that the load call will only search in this specific DLC content.
		// You can leave this as null to search in all loaded DLC's
		string dlcUniqueKey = null;

		// Load an asset with the following path as a game object
		// Note that the DLC may be loaded as part of this call if it is not already loaded into memory, assuming that the containing DLC can be determined from the asset load path.
		// Of course this load call will fail if the DLC is not owned by the user according to DRM, or if it cannot be loaded for whatever reason
		DLCAsync<GameObject> request = DLC.LoadAssetAsync<GameObject>("Prefabs/Truck.prefab", dlcUniqueKey);

		// Wait for load to complete
		yield return request;

		// Check for found
		Debug.Log("Load successful: " + request.Successful);
		Debug.Log("Load status: " + result.Status);
		Debug.Log("Loaded asset: " + request.Result);
	}
}
```
