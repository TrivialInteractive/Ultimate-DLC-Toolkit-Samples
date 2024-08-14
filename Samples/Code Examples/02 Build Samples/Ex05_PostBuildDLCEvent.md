This example shows how you can receive a callback via an editor script when the DLC build pipeline has finished building any DLC content. You can use this event to run custom code immediatley after the DLC build pipeline finishes, regardless of the build result.
```cs
using UnityEngine;
using DLCToolkit;
using DLCToolkit.BuildTools;

// This attribute registers the script to receive post build events from the DLC build pipeline
[DLCPostBuildAttribute]
public class Example : DLCBuildEvent
{
        // Called by the DLC build pipeline just after a DLC build has finished
        public override void OnBuildEvent()
        {
                Debug.Log("DLC build has finished!");
        }
}
```
