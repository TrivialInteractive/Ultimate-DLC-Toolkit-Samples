This quick example shows how to list all published DLC that are available for the game using the DRM provider if available. 
Using this method requires a DRM provider such as Steamworks to be setup (More info in user guide), in which case all published DLC unique keys will be listed even if the user has not purchased or subscribed.
Note that some DRM platforms do not support listing all published DLC content, but there is another option shown below in that case.

```cs
using System.Collections;
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	IEnumerator Start()
	{
		// Try to get all published DLC unique keys from the DRM provider.
		DLCAsync<string[]> request = DLC.RemoteDLCUniqueKeysAsync;

		// Wait for request to complete
		yield return request;

		// Check for successful
		if(request.Success == false)
		{
			Debug.LogError("An error occurred while listing the published dlc keys");
			return;
		}

		// Report all unique keys
		foreach(string uniqueKey in request.Result)
			Debug.Log("Unique Key = " + uniqueKey);
	}
}
```

As mentioned above, not all DRM providers will support listing published unique keys.
In such a case (Where the above call to `RemoteDLCUniqueKeysAsync` would throw a `NotSupportedException`) you can use the local alternative which will be able to list unique keys for all DLC content at the time of building the game.
It is possible for new DLC content to be published and for the game to not be aware of it, so you should be aware that this approach may not find all DLC if the game is not fully up to date.

```cs
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	void Start()
	{
		// Try to get all local DLC unique keys that were available when this version of the game was built
		string[] uniqueKeys = DLC.LocalDLCUniqueKeys;

		// Report all unique keys
		foreach(string uniqueKey in uniqueKeys)
			Debug.Log("Unique Key = " + uniqueKey);
	}
}
```