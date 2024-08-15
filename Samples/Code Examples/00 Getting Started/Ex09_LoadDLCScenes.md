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

Scenes can also be loaded asynchronously which is recommend in most cases, especially for loading additional game maps or levels that may take some time to load. We need to specify whether we want to load the scene using single or additive mode, as per Unity standard scene loading. Loading is performed on a background thread and you can wait for that to complete in a coroutine or similar:

```cs
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	void Start()
	{
		// This example assumes that we have already found the scene to load via one of the above approaches
		DLCSceneAsset sceneToLoad = ...

		// Load async will return an awaitable object
		DLCAsync async = sceneToLoad.LoadAsync(LoadMode.Single);

		// Wait for the load to complete - the game can continue running in the meantime
		yield return async;

		// Check for successful - The scene will now be loaded at this stage
		if(async.IsSuccessful == true)
		{
			Debug.Log("Loaded DLC Scene: " + sceneToLoad.Name);
		}
	}
}
```

Additionally, scenes can be loaded in the background but not activated until required by game logic. This is essentially equivilent to Unity's `allowSceneActivation` option, but has a slightly different workflow. Firstly you must specify that scene activation should be postponed when making the initial load call, and then it is just a case of manually activating the scene at a later time via the same `DLCAsync` reference:

```cs
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	// Store a reference to the async scene load object
	DLCAsync asyncScene = null;

	void Start()
	{
		// This example assumes that we have already found the scene to load via one of the above approaches
		DLCSceneAsset sceneToLoad = ...

		// Load async will return an awaitable object
		bool allowSceneActivation = false;
		asyncScene = sceneToLoad.LoadAsync(LoadMode.Single, allowSceneActivation);

		// Wait for the load to complete - the game can continue running in the meantime
		yield return asyncScene;

		// Check for successful - The scene loading will be completed at this stage, but the scene will not be active in the game
		if(asyncScene.IsSuccessful == true)
		{
			Debug.Log("Loaded DLC Scene Pending Activation: " + sceneToLoad.Name);
		}
	}

	// For this example we assume that this method is bound to some UI button that should be clicked when a scene loading screen has finished for example
	public void OnStartGameClicked()
	{
		// Now we can inform DLC toolkit that the scene should be activated, meaning Unity will switch to it
		if(asyncScene != null && asyncScene.IsDone == true)
			asyncScene.ActivatePendingScene();
	}
}
```

One more useful tip is that the loading progress can be accessed if required to show a loading/progress bar or similar in your game UI. It only requires minor changes to how the async operation is awaited, and then you will be able to access progress values and status messages in realtime to give user feedback:

```cs
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	void Start()
	{
		// Before load code omitted for this example

		DLCAsync async = ...

		// Instead of yielding the object directly, we can create our own loop that runtime until loading has finished, and yield per frame instead
		while(async.IsDone == false)
		{
			// Report progress percentage - A normalized 0-1 progress is also available, as well as status message
			Debug.LogFormat("Loading Scene = {0}%", async.ProgressPercentage);

			// IMPORTANT - We need to yield every cycle of the loop to avoid freezing the game or crashing the editor
			yield return null;
		}

		// After load code omitted for this example
	}
}
```
