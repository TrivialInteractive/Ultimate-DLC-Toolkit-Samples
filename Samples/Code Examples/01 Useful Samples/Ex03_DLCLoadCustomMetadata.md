This short example shows a couple of ways that you can access custom metadata via the runtime API. In order to access the custom metadata element of some DLC content, you first need to have access to the `IDLCMetadata` for that particular DLC. There are a couple of ways you can access that by loading the DLC content either partially or fully.

##### Metadata Only Load
Metadata only load mode is intended as a quick way to fetch the metadata of DLC content that has not been loaded into memory already. This mode supports loading DLC partially from various locations, including DRM providers if required:
```cs
// Fetch the metadata only for the DLC - This allows quick access to metadata, only loading the bare minimum required from the DLC.
IDLCMetadata metadata = DLC.LoadDLCMetadataFrom("DLC/Example DLC 1.dlc");

// Load via the DLC unique key which will use DRM to access the content - Steam DLC AppID used as an example, but typically the unique key may not be known at compile time
metadata = DLC.LoadDLCMetadata("2683920");
```

##### Full Load
Alternativley if you have already loaded the DLC content into memory via one of the many load methods, you will have a reference to a `DLCContent` instance which can provide access to the metadata for the DLC:
```cs
// Load the DLC as normal - Typically you will probably not use `LoadFrom`, but it is suitable for this example
DLCContent dlc = DLC.LoadDLCFrom("DLC/Example DLC 1.dlc");

// Now we can access the metadata while the DLC content remains loaded (Until `dlc.Unload()` is called)
IDLCMetadata metadata = dlc.Metadata;
```

##### Access Custom Metadata
Once we have gained access to the `IDLCMetadata` for the DLC content using one of the above approaches, we can now fetch the custom metadata element using the following methods (This example assumes that a custom metadata type named `ExampleMetadata` has been created (As per the previous sample), assigned and built as part of the DLC): 
```cs
// Fetch metadata element using one of the above approachs
IDLCMetadata metadata = ...

// First we can check if there is any custom metadata included in the DLC
if(metadata.HasCustomMetadata == true)
  Debug.Log("Found custom metadata!");

// Access the DLC and convert to the metadata type
ExampleMetadata customMetadata = metadata.GetCustomMetadata() as ExampleMetadata;

// Alternatively the generic method may be more ideal
customMetadata = metadata.GetCustomMetadata<ExampleMetadata>();
```

From there you now have full access to all the custom metadata fields you created, assuming they we made public or have been exposed by an API or property:
```cs
// Access custom metadata as shown above
ExampleMetadata customMetadata = ...

// Now we can use the public API to get data stored in serialized fields
Debug.Log(customMetadata.myStringField);
Debug.Log(customMetadata.myDoubleArray);
```
