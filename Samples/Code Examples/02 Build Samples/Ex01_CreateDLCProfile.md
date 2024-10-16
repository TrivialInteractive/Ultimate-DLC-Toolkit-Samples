It is possible to create DLC Profile assets using the Ultimate DLC Toolkit build pipeline from an editor script, which may be helpful for advanced users or for automation purposes.
Creating a profile from code can be done as shown below, and the DLC name and folder are required for that:
```cs
using DLCToolkit.BuildTools;
using DLCToolkit.Profile;

public class Example
{
        static void CreateDLCProfile()
        {
                // Create the DLC profile asset in the project
                // The profile folder must be relative to the Unity project
                DLCProfile createdProfile = DLCBuildPipeline.CreateDLCProfile("Assets/DLC", "MyDLCName");
        }
}
```
You can also create a `DLCProfile` scriptable object manually and then save that in the project using the build API:
```cs
using UnityEngine;
using DLCToolkit.BuildTools;
using DLCToolkit.Profile;

public class Example
{
        static void CreateDLCProfile()
        {
                // Create DLC profile scriptable object
                DLCProfile profile = ScriptableObject.CreateInstance<DLCProfile>();
                profile.DLCName = "MyDLCName";

                // Create the DLC profile asset in the project
                DLCBuildPipeline.CreateDLCProfile(profile, "Assets/DLC");
        }
}
```
