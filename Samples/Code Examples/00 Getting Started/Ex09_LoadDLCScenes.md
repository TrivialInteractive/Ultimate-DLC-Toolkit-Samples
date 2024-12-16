This quick example shows how to load scene assets from a given DLC content.
Scene assets are just Unity scenes and can be used to add new maps/levels to your game via DLC (Map packs for example)

Load a scene where the name is known ahead of time. May not be useful in most cases but could be suitable if all DLC include some predefined named asset like `index.asset` for example:

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
using DLCToolkit.Assets;

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
using System.Collections;
using UnityEngine;
using DLCToolkit;
using DLCToolkit.Assets;

public class Example : MonoBehaviour
{
	IEnumerator Start()
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
using System.Collections;
using UnityEngine;
using DLCToolkit;
using DLCToolkit.Assets;

public class Example : MonoBehaviour
{
	// Store a reference to the async scene load object
	DLCAsync asyncScene = null;

	IENumerator Start()
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
using System.Collections;
using UnityEngine;
using DLCToolkit;
using DLCToolkit.Assets;

public class Example : MonoBehaviour
{
	IEnumerator Start()
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

#### Load Scene From Any DLC (1.2 or newer)
As of version 1.2, it is now possible to load scene assets from any loaded DLC by providing the name or path of the scene. It is also possible in the case of async calls to load the DLC on demand as part of the same load call, in a similar way that addressables works.

Firstly the non-async variants only support loading scenes from DLC contents that were already loaded before the call, but it can make things easier to manage since you don't have to deal with finding the correct DLCContent and DLCSceneAsset before attempting to load, as this example shows:

```cs
using UnityEngine;
using DLCToolkit;
using DLCToolkit.Assets;

public class Example : MonoBehaviour
{
	void Start()
	{
		// Load some DLC first
		DLC.LoadDLC("MyDLCKey");

		// Note that you can optionally provide a hint DLC unique key so that the load call will only search in this specific DLC content.
		// You can leave this as null to search in all loaded DLC's
		string dlcUniqueKey = null;

		// Load a scene with the following path as a game object
		// Note that the DLC containing the scene must be loaded before this call, but check out the async alternative to support on demand DLC loading
		bool didLoad = DLC.LoadScene("Scenes/Level1", LoadSceneMode.Single, dlcUniqueKey);

		// Check for found
		Debug.Log("Loaded scene: " + didLoad);
	}
}
```

Finally you can use a similar API to load the scene asynchronously. Note that the async variants differ slightly because they support loading the DLC content on demand if it is not already loaded into memory. In that regard it works in a similar way to addressables, so may be useful to users who are switching to DLC Toolkit so that they can keep a similar code base.

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

		// Should the scene be activated as soon as loading has finished.
		// Can be disabled in the case that you want to show a loading screen or similar, and you can manually activate the scene later as shown below
		bool allowSceneActivation = true;

		// Load a scene with the following path as a game object
		// Note that the DLC may be loaded as part of this call if it is not already loaded into memory, assuming that the containing DLC can be determined from the scene load path.
		// Of course this load call will fail if the DLC is not owned by the user according to DRM, or if it cannot be loaded for whatever reason
		DLCAsync request = DLC.LoadSceneAsync("Scenes/Level1", LoadSceneMode.Single, dlcUniqueKey);

		// Wait for load to complete
		yield return request;

		// Check for found
		Debug.Log("Load successful: " + request.Successful);
		Debug.Log("Load status: " + result.Status);

		// Allow the loading screen to be displayed for some time
		yield return new WaitForSeconds(2f);

		// Manually activate the scene (Switch to this scene)
		if(allowSceneActivation == false)
		{
			// First we need a reference to the scene asset
			DLCSceneAsset sceneAsset = DLC.FindScene("Scenes/Level1", dlcUniqueKey);

			// Now we can manually activate the scene
			if(sceneAsset != null)
				sceneAsset.ActivatePendingScene();
		}
	}
}
```
