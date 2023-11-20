This quick example shows how to load DLC content from an arbitrary file path asynchronously. 
For most DLC cases this method will usually not be suitable, but it can be useful for loading add-on packs or other DLC content that should be installed by the user.
This method will load the DLC content in the background allowing the main game thread to continue in order to display a loading screen for example.

```cs
using System.Collections;
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	// The path where dlc will be loaded from
	// Must be a valid dlc content file
	public string dlcPath = "DLC/Expansion1.dlc";

	IEnumerator Start()
	{
		// Load the dlc from path
		DLCAsync<DLCContent> request = DLC.LoadDLCFromAsync(dlcPath);

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