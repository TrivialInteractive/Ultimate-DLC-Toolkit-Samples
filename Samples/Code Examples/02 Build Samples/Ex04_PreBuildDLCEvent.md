This example shows how you can receive a callback via an editor script when the DLC build pipeline begins building any DLC content. You can use this event to run custom code just before the DLC build pipeline begins.
```cs
using UnityEngine;
using DLCToolkit;
using DLCToolkit.BuildTools;

// This attribute registers the script to receive pre build events from the DLC build pipeline
[DLCPreBuildAttribute]
public class Example : DLCBuildEvent
{
        // Called by the DLC build pipeline just before any DLC content build is started
        public override void OnBuildEvent()
        {
                Debug.Log("DLC build has started!");
        }
}
```
