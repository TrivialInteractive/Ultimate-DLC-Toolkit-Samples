This quick example shows how to load DLC content from an arbitrary file path. For most DLC cases this method will usually not be suitable, but it can be useful for loading add-on packs or other DLC content that should be installed by the user.

```cs
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	// The path where dlc will be loaded from
	// Must be a valid dlc content file
	public string dlcPath = "DLC/Expansion1.dlc";

	void Start()
	{
		// Load the dlc from path
		DLCContent content = DLC.LoadDLCFrom(dlcPath);

		// Check for loaded
		if(content.IsLoaded == false)
			Debug.LogError("Failed to load DLC");
	}
}
```