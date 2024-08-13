This short example shows how to create a simple custom metadata class that can be used by Ultimate DLC Toolkit to store, edit and load custom metadata as part of your DLC content. Once the class has been created, it will need to be assigned to a DLC Profile asset in the editor before cusotm metadata can be used. More info in the user guide.

_Note: The custom metadata type must be declared inside the default Assembly-CSharp or a runtime assembly if using assembly definitions. If you are using assembly definitions, you will also need to add a guid reference to `DLCToolkit.asmdef` in order to have access to `DLCToolkit.DLCCustomMetadata` type._
```cs
using System;
using DLCToolkit;

[Serializable]
public class ExampleMetadata : DLCCustomMetadata
{
  [Serializable]
  class Nested
  {
    public string nestedField;
  }

  // ### The following fields WILL be serialized ###
  public string myStringField = "hello";

  [SerializeField]
  private int myNumberField = 1234;

  [SerializeField]
  internal Vector3 myVectorField = new Vector3(1.0f, 1.5f, 2.0f);

  public double[] myDoubleArray;

  [SerializeField]
  protected List<Vector2> myVector2List;

  [SerializeField]
  private Nested nested;


  // ### The following fields WILL NOT be serialized ###
  public UnityEngine.Object myAssetReference;

  [SerializeField]
  private UnityEngine.Texture2D[] myTextureArray;
}
```

As you can see, all primitive types except `System.Object` are supported, plus arrays and lists of supported element types, as well as a few Unity structs that you might expect.

Serialization follows standard Unity serialization principles in most cases. For example: Public fields are serialzied automatically, non-public fields can be serialized using the `SerializeField` attribute and so on. Additionally all the usual editor attributes such as `Range` or `Tooltip` will work correctly when editing the custom metadata.

##### Built in types:
`bool, byte, sbyte, char, decimal, double, float, int, uint, nint, nuint, long, ulong, short, ushort, string`

##### Unity Types:
`Vector2, Vector3, Vector4, Quaternion, Color, Color32, plus some others`

##### Custom Serializable types:
`Classes or structs not deriving from UnityEngine.Object and with the [System.Serializable] attribute are supported`

##### Arrays/Collections:
`System.Array or System.List<> where the element type is any supported type listed (Including nested collections for example)`
