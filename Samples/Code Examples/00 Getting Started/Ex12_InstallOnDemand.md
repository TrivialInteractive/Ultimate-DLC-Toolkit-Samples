This quick example shows how you can request that a specific DLC content is installed on demand. You need to provide the DLC unique key of the DLC you want to install.
Not all DRM providers will support installing on demand, in which case the request would fail with some error status. The DLC will need to be owned or added to the users account in order for the DRM service to commence installation of the DLC, otherwise the install request will likley fail with an error status. 
Installation may take some time to complete, but usually if the game exists during an install request, the DRM service (Steam, Google Play) will continue with the download/installation until completed, and next time the game launches the DLC will now be available.

```cs
using System.Collections;
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	IEnumerator Start()
	{
		// Create the install request - use Steam AppId DLC key as the unique key for this example
		DLCAsync request = DLC.RequestInstall("2683920");

		// Wait for the install to complete - other code can run in the meantime as this does not block the game
		yield return request;

		// Check for installed
		if(request.IsSuccessful == true)
		{
			Debug.Log("DLC was installed!");
		}
		else
		{
			Debug.LogError("DLC could not be installed: " + request.Status);
		}
	}
}
```
