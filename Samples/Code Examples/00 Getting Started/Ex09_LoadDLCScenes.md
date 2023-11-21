This quick example shows how to load scene assets from a given DLC content.
Scene assets are just Unity scenes and can be used to add new maps/levels to your game via DLC (Map packs for example)

Load a scene where the name is known ahead of time. May not be useful in most cases but could be suitable if all DLC include some predefined named asset like `index.asset` for example:

```cs
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	void Start()
	{
		// For this example we assume that the DLC content has already been loaded
		DLCContent content = ...

		// Try to find the scene with the name
		// DLCSceneAsset provides lots of useful metadata such as path, extension, type and more
		DLCSceneAsset asset = content.SceneAssets.Find("Test.unity");

		// Check for found
		if(asset == null)
		{
			Debug.LogError("Asset not found: Test.unity");
			return;
		}

		// Load the actual scene
		asset.Load(LoadSceneMode.Single);
	}
}
```

In most cases it is likely that you will not know the name of a scene ahead of type, or you may want to design your game/DLC in such a way that scene name is not important. 
To help with this case, DLC Toolkit provides a number of methods that can help find all included scenes.
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

		// Find all scenes in the specified folder (relative to the DLC root folder)
		foreach(DLCSceneAsset asset in content.SceneAssets.FindAllInFolder("Scenes/Maps"))
		{
			Debug.Log("Game Map: " + asset.RelativeName);
		}

		// Find all scenes in the DLC
		foreach(DLCSceneAsset asset in content.SceneAssets.FindAll())
		{
			Debug.Log("Game Map: " + asset.RelativeName);
		}
	}
}
```