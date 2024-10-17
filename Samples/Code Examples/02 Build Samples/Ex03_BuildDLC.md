For advanced users and automation purposes, Ultimate DLC Toolkit offers full build support from an editor script if you need to build DLC from custom tools or as part of a custom build pipeline. 
The DLC build pipeline is a batched build process, meaning that any number of DLC profiles can be built as part of a single operation, for any number of platforms, or you can just build a single DLC profile depending upon your needs. For that reason there are a number of build API's to consider so that we do not place any limitations on what ou can achieve via editor scripting.
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
