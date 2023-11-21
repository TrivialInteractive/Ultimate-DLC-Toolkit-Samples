This quick example shows how to load DLC content with the provided unique key.
Note that the DLC must be purchased and installed locally (IsAvailable = true) in order for the load to be successful.
Using this method requires a DRM provider such as Steamworks to be setup (More info in user guide), in which case the DRM provider will usually handle things like auto-delivery of content when purchased by the user.

```cs
using System.Collections;
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	// The unique key for the DLC setup in editor
	public string dlcKey = "dlc_expansion_1";

	IEnumerator Start()
	{
		// Load the dlc with the unique key
		DLCAsync<DLCContent> request = DLC.LoadDLCAsync(dlcKey);

		// Check for load to complete
		while(request.IsDone == false)
		{
			// Report load progress
			Debug.Log("DLC Load Progress = " + request.Progress);

			// Wait a frame
			yield return null;
		}

		// Check for successful
		if(request.IsSuccessful == false)
		{
			Debug.LogError("Failed to load DLC");
		}
		else
		{
			// Get the DLC content where we can load assets etc.
			DLCContent content = request.Result;
		}
	}
}
```