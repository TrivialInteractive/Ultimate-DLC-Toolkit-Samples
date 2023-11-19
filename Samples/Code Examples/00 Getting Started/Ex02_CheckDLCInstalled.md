This quick example shows how to check if a DLC with the given unique key has been purchased by the user and is installed locally.
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
		// Check if the dlc is available - requires a DRM provider to be available for the current platform such as Steamworks
		DLCAsync<bool> request = DLC.IsAvailable(dlcKey);

		// Wait for request to complete
		yield return request;

		// Check for successful
		if(request.Success == false)
		{
			Debug.LogError("An error occurred while checking the dlc status");
			return;
		}

		// Check for available
		Debug.Log("Available = " + request.Result);
	}
}
```