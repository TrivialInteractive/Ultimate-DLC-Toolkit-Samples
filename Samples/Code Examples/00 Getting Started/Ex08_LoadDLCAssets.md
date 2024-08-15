This quick example shows how to load shared assets from a given DLC content.
Shared assets are non-scene and non-script assets such as prefabs, material, textures, scriptable object assets, and much more.  

Load an asset where the asset name is known ahead of time. May not be useful in most cases but could be suitable if all DLC include some predefined named asset like `index.asset` for example:

```cs
using UnityEngine;
using DLCToolkit;

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
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	void Start()
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
