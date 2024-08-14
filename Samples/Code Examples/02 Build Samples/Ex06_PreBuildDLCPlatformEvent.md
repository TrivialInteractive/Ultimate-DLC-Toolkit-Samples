This example shows how you can receive a callback via an editor script when the DLC build pipeline begins building DLC content for a specific platform. This even may be called a number of times during the build, passing in each DLC profile and platform profile that is about to be built.

If you start building a DLC profile with Windows, Linux and OSX platforms enabled, then this event will be called 3 times, once for each platform. The `DLCProfile` is the profile asset for the DLC that is being built, and the `DLCPlatformProfile` represents the current platform that is being built. 
```cs
using UnityEngine;
using DLCToolkit;
using DLCToolkit.BuildTools;
using DLCToolkit.BuildTools.Events;

// This attribute registers the script to receive pre build events from the DLC build pipeline
[DLCPreBuildPlatformProfileAttribute]
public class Example : DLCBuildPlatformProfileEvent
{
        // Called by the DLC build pipeline just before a platform build begins
        public override void OnBuildProfileEvent(DLCProfile profile, DLCPlatformProfile platformProfile)
        {
                Debug.Log("Start building DLC: " + profile.DLCName);
                Debug.Log("For target platform: " + platformProfile.PlatformFriendlyName);
        }
}
```
