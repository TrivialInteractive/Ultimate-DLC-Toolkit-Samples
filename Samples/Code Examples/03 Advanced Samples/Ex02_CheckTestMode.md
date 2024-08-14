This example will show how you can check whether Ultimate DLC Toolkit is currently in editor test mode. Editor test mode is a means of simulating DRM management in the editor for quick and easy testing of DLC content and API usage.
```cs
using UnityEngine;
using DLCToolkit;

class Example : MonoBehaviour
{
  void Start()
  {
    Debug.Log("Test Mode Enabled = " + DLC.IsTestMode);
  }
}
```
Editor test mode is enabled by default and will display a message in the console when entering play mode to make that evident. Additionally Test Mode can be disabled or enabled at any time via the `Tools` menu.
