This quick example shows how to load a batch of DLC content in parallel as a single operation. Batch loading is highly recommended over sequential loading, as the total load time required is usually only the time taken to load the largest DLC, plus some additional minor overhead for async and batch management.
 Batch loading is recommended for loading all DLC content owned by the user at the start of the game, and is only available in async variants (Async loading is always recommended in any case).

```cs
using System.Collections;
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	string[] dlcPaths = new string[] { "DLC/Example1.dlc", "DLC/Example2.dlc", "DLC/Example3.dlc" };

	IEnumerator Start()
	{
		// Create the load request
		DLCBatchAsync<DLCContent> request = DLC.LoadDLCBatchFromAsync(dlcPaths);

		// We can wait for the request to complete using `yield return request`, however you can also get progress updates if required
		while(request.IsDone == false)
		{
			// Report the progress percentage - there is also `.Progress` containing a normalized 0-1 value
		    	Debug.LogFormat("Load progress: {0}%", request.ProgressPercentage);

			// VERY IMPORTANT - We must yield at each iteration of the loop to avoid blocking the main thread which would freeze the game or crash the editor
		    	yield return null;
		}

		// Now we can check the result - Successful will only be true if all operations completed successfully
		Debug.Log("Success = " + request.IsSuccessful);

		// For better status information we can check how many operations were successful
		Debug.LogFormat("Success = {0}/{1}", request.CompletedCount, request.TotalCount);

		// We can get access to the successfully loaded contents list so that we can access metadata and assets
		// Content that failed to load will not be included here
		DLCContent[] contents = request.Result;
		
		// Todo: Do something with contents...	


		// If we need more information about why the failed DLC could not be loaded, we can access the status of all failed tasks
		IEnumerable<DLCAsync<DLCContent>> failed = request.Tasks.Where(t => t.IsSuccessful == false);

		// Then we can report the status which will include the reason for the failure
		foreach(DLCAsync<DLCContent> failedLoad in failed)
			Debug.LogError("Could not load DLC: " + failedLoad.Status);
	}
}
```

There is also full support for batch loading using the DRM API - Ie. supplying DLC unique keys for the load request which will be resolved and checked for ownership as part of the load request using the setup DRM provider if available. The load request is issues slightly differently, but checking and accessing the load result and status is identical to the above example, so have been ommitted here.

```cs
using UnityEngine;
using DLCToolkit;

public class Example : MonoBehaviour
{
	// Using Steam App Id unique keys here for Steam DRM
	string[] dlcUniqueKeys = new string[] { "2683113", "2683920", "2683952" }; 

	void Start()
	{
		// Create the load request
		DLCBatchAsync<DLCContent> request = DLC.LoadDLCBatchAsync(dlcUniqueKeys);
	}
}
```
