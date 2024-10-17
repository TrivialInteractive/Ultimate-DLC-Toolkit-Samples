When working with DLC content from an editor script, a common requirement is to be able to find all DLC profiles that are part of the project. You could use the Unity asset database as these assets are simply scriptable objects, but we have also included API's to help with that and offer filtering in addition to search by platform for example.
Firstly you can find a specific DLC profile if you already know the DLC profile name you are looking for, or the unique key:
```cs
using DLCToolkit.BuildTools;
using DLCToolkit.Profile;
using System;

public class Example
{
        static void FindDLCProfile()
        {
                // Find the DLC profile with the specified name and version
                string dlcName = "My DLC 1";
                Version dlcVersion = null;

                // Try to find the profile by name
                DLCProfile foundProfile = DLCBuildPipeline.GetDLCProfile(dlcName, dlcVersion);

                // Find the DLC profile by unique key
                string dlcUniqueKey = "my_dlc_key";

                // Try to find the profile by unique key
                foundProfile = DLCBuildPipeline.GetDLCProfile(dlcUniqueKey);
        }
}
```
Additionally you can find all DLC profiles that have been created in the project, with optional filtering if required:
```cs
using DLCToolkit.BuildTools;
using DLCToolkit.Profile;

public class Example
{
        static void FindDLCProfile()
        {
                // Find all DLC profiles - Note: will only find enabled profiles
                DLCProfile[] allProfiles = DLCBuildPipeline.GetAllDLCProfiles();

                // To find disabled profiles also, simply pass `true` for the second argument
`                DLCProfile[] allProfilesIncludingDisabled = DLCBuildPipeline.GetAllDLCProfiles(null, true);

                // Finally we can find only DLC profiles that are setup to build for a specific platform.
                // Simply specify the build target(s) we are interested in, and then all profiles setup for those platforms will be found. This search is inclusive, so if you provide multiple built targets to search for, a profile will be matched if it is ony sertup for one of those build targets.
                BuildTarget[] platforms = new BuildTarget[]
                {
                        BuildTarget.StandaloneWindows64,
                        BuildTarget.StandaloneLinux64,
                        BuildTarget.StandaloneOSX,
                };

                DLCProfile[] allProfilesFiltered = DLCBuildPipeline.GetAllDLCProfiles(platforms, false);
        }
}
```
