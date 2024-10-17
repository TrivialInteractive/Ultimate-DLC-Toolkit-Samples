For advanced users and automation purposes, Ultimate DLC Toolkit offers full build support from an editor script if you need to build DLC from custom tools or as part of a custom build pipeline. 
The DLC build pipeline is a batched build process, meaning that any number of DLC profiles can be built as part of a single operation, for any number of platforms, or you can just build a single DLC profile depending upon your needs. For that reason there are a number of build API's to consider so that we do not place any limitations on what you can achieve via editor scripting.
Firstly you can build all enabled DLC profiles in the project with a single call:
```cs
using DLCToolkit.BuildTools;
using DLCToolkit.Profile;

public class Example
{
        static void BuildDLC()
        {
                // Build all DLC profiles in the project and get the build result
                DLCBuildResult result = DLCBuildPipeline.BuildAllDLCContent();

                // You can build to a specific output folder if required, otherwise the DLC will be built in the location specified by the DLC profile asset
                result = DLCBuildPipeline.BuildAllDLCContent("C:/DLC/Build");

                // Additionally you can specify which platforms should be built. Useful if you want to test on a speciifc platform as you can save time by only building for that platform
                BuildTarget[] platforms = new BuildTarget[]
                {
                        BuildTarget.StandaloneWindows64,
                };

                result = DLCBuildPipeline.BuildAllDLCContent(null, platforms);
        }
}
```
You can also build only the specified DLC profiles if you want to build a subset of the project DLC content. You can use the previous example `GetDLCProfiles` to find the DLC profile assets you are interested in:
```cs
using DLCToolkit.BuildTools;
using DLCToolkit.Profile;

public class Example
{
        static void BuildDLC()
        {
                // Assumes you already have found the profiles you want to build
                DLCProfile[] buildProfiles = ...

                // Build the specified profiles only
                DLCBuildResult result = DLCBuilePipeline.BuildDLCContent(buildProfiles);

                // We can also specifiy an override build location and built targets for this overload also
                BuildTarget[] platforms = new BuildTarget[]
                {
                        BuildTarget.StandaloneWindows64,
                };

                result = DLCBuildPipeline.BuildDLCContent(profiles, "C:/DLC/Build", platforms);
        }
}
```
There are also convenience API's for building a single DLC profile asset: 
```cs
using DLCToolkit.BuildTools;
using DLCToolkit.Profile;

public class Example
{
        static void BuildDLC()
        {
                // Assumes you already have found the profile you want to build
                DLCProfile buildProfile = ...

                BuildTarget[] platforms = new BuildTarget[]
                {
                        BuildTarget.StandaloneWindows64,
                };

                // Build the specified profile only
                DLCBuildResult result = DLCBuilePipeline.BuildDLCContent(buildProfile, null, platforms);
        }
}
```
Finally there are special API's for working wth DLC content that is setup as `Ship With Game`, meaning that the content is intended to be distributed with the game executable files/content. These API's run automatically as part of Unity's standard player build pipeline so that the DLC content gets build and packaged with the game automatically at the time of building, but you can also ru them manually if you are using a custom build API or similar. These API's will essentially build the necessary `Ship With Game` DLC content, and then package it with the built game. For that reason you must specifiy the build target when calling these API's:
```cs
using DLCToolkit.BuildTools;
using DLCToolkit.Profile;

public class Example
{
        static void BuildShipWithGameDLC()
        {
                // Build the ship with game content and install it
                DLCBuildResult result = DLCBuildPipeline.BuildAllDLCShipWithGameContent(BuildTarget.StandaloneWindows64);

                // Note that the above call will only install content that is setup as `Ship With Game: Streaming`. To install other content we also need to install the build directory where the standalone build will be created by Unity
                result = DLCBuildPipeline.BuildAllDLCShipWithGameContent(BuildTarget.StandaloneWindows64, true, "Build/Windows");

                // Finally we can disable the install aspect if required, so that the DLC content is only built but not packaged with the game. Maybe useful if you want to install the content manually as part of a custom build pipeline or similar
                bool install = false;

                result = DLCBuildPipeline.BuildAllDLCShipWithGameContent(BuildTarget.StandaloneWindows64, install, "Build/Windows");
        }
}
```
#### Build Options
All of the mentioned build API's also support an additiona build options argument if you need more control over the build. The following options can be specified and combined as flags to change the build behaviour:
```cs
using DLCToolkit.BuildTools;
using DLCToolkit.Profile;

public class Example
{
        static void BuildOptions()
        {
                DLCBuildOptions options = DLCBuildOptions.None;

                // Forces the build pipeline to also build DLC content that is disabled in the DLC profile asset
                options |= DLCBuildOptions.IncludeDisabledDLC;

                // Ultimate DLC Toolkit uses incremental building to speed up subsequent builds. You can force all data to be rebuilt with the following option, build the build may take significantly longer
                options |= DLCBuildOptions.ForceRebuild;

                // Ultimate DLC Toolkit supports adding additionl scripting content that was not part of the base game on some Mono platforms. You can disable that feature support using this option (Note that sciprts used from the base game will still work just fine in the DLC)
                options |= DLCBuildOptions.DisableScripting;

                // Force new DLC scripts to be compiled in debug mode with symbols to support debugging (Experimental)
                options |= DLCBuildOptions.DebugScription;


                // These build options can be provided to any build API, for example:
                DLCBuildPipeline.BuildAllDLCContent(null, null, options);
        }
}
```
#### Build Result
All of the mentioned build API's also return a `DLCBuildResult` object containing detailed information about the build that was requested, including information about successful and failed builds.
```cs
using DLCToolkit.BuildTools;
using DLCToolkit.Profile;

public class Example
{
        static void BuildResult()
        {
                // Assumes that the build result has been returned by one of the build API's mentioned above
                DLCBuildResult result = ...

                // Firstly we can check whether all builds were successful, since the build API is batched it is normally dealing with multiple builds as a single request.
                Debug.Log("Successful: " + result.AllSuccessful);

                // We can check how many builds were successful and how many failed (Note that there is 1 build per profile per platform)
                Debug.Log("Successful Count: " + result.BuildSuccessCount + ", Failed Count: " + result.BuildFailedCount);

                // There is also information available about the build start time and duration
                Debug.Log("Build Start Time: " + result.BuildStartTime);
                Debug.Log("Build Elapsed Time: " + result.ElapsedBuildTime);
        }
}
```
Next we can work with the `DLCBuildTask` objects included in the result, which each represent a specific build request for a specific platform.
```cs
using DLCToolkit.BuildTools;
using DLCToolkit.Profile;

public class Example
{
        static void BuildTaskResult()
        {
                // Assumes that the build result has been returned by one of the build API's mentioned above
                DLCBuildResult result = ...

                // We can find all successful build tasks as shown
                foreach(DLCBuildTask task in result.GetSuccessfulBuildTasks())
                        Debug.Log("Profile: " + task.Profile.DLCUniqueKey + ", Platform: " + task.PlatformProfile.PlatformFriendlyName);

                // In a similar way we can also find all failed build tasks
                foreach(DLCBuildTask task in result.GetFailedBuildTasks())
                        Debug.Log("Profile: " + task.Profile.DLCUniqueKey + ", Platform: " + task.PlatformProfile.PlatformFriendlyName);
        }
}
```
Once we have a reference to a `DLCBuildTask` object, we can then examine it further to get additional information:
```cs
using DLCToolkit.BuildTools;
using DLCToolkit.Profile;

public class Example
{
        static void BuildTaskResult()
        {
                // Assumes that the build task reference has been acquired as shown in the above example
                DLCBuildTask task = ...

                // We can check if the build was successful, but we probably already know that at this point
                Debug.Log("Successful: " + task.Success);

                // We can access the DLC profile that was used for this build request, as well as the specfic platform profile
                DLCProfile profile = task.Profile;
                DLCPlatformProfile platformProfile = task.PlatformProfile;

                Debug.Log("Build for: + platformProfile.PlatformFriendlyName);

                // If the build was successful, we can get the otput file path of the built DLC content
                Debug.Log("OutputPath: " + task.OutputPath;

                // Finally we can also get the specific timings for this particular build task
                Debug.Log("Build Start Time: " + task.BuildStartTime);
                Debug.Log("Build Elapsed Time: " + task.ElapsedBuildTime);
        }
}
```
