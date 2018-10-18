## Uploading Files and Graphics
Graphics allow for you to better customize what you would like to have your users see and provide a better User Interface.

When developing an application using SmartDeviceLink, two things must always be remembered when using graphics:

1. You may be connected to a head unit that does not display graphics.
2. You must upload them from your mobile device to Core before using them.

!!! note
Many of these features will be handled for you automatically by the `ScreenManager` and other managers. This guide will be for using the `FileManager` directly through `SdlManager`
!!!

### Detecting if Graphics are Supported
Being able to know if graphics are supported is a very important feature of your application, as this avoids you uploading unneccessary images to the head unit. In order to see if graphics are supported, use the `getCapability()` method of a valid `sdlManager.getSystemCapabilityManager()` to find out the display capabilities of the head unit.

```java
sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.DISPLAY, new OnSystemCapabilityListener(){

   @Override
   public void onCapabilityRetrieved(Object capability){
      DisplayCapabilities dispCapability = (DisplayCapabilities) capability;
   }

   @Override
   public void onError(String info){
      Log.i(TAG, "Capability could not be retrieved: "+ info);
   }
 });
```

### SDL File and SDL Artwork

New to version 4.7 of the SDL Android library are `SdlFile` and `SdlArtwork` objects. These have been created in parallel with the `FileManager` to help streamline SDL workflow. `SdlArtwork` is an extension of `SdlFile` that pertains only to graphic specific file types, and its use case is similar. For the rest of this document, `SdlFile` will be described, but everything also applies to `SdlArtwork`.

#### Creation

One of the hardest parts about getting a file into SDL was the boilerplate needed to convert the file into a byte array that was used by the head unit. Now, you can instantiate an `SdlFile` with:

##### A resource ID
```java
new SdlFile(@NonNull String fileName, @NonNull FileType fileType, int id, boolean persistentFile)
```
##### A URI
```java
new SdlFile(@NonNull String fileName, @NonNull FileType fileType, Uri uri, boolean persistentFile)
```
And last but not least

##### A byte array
```java
new SdlFile(@NonNull String fileName, @NonNull FileType fileType, byte[] data, boolean persistentFile)
```

without the need to implement the methods needed to do the conversion of data yourself.

### Uploading a File 

Uploading a file with the `FileManager` is a simple process. With an instantiated `SdlManager`,
you can simply call:

```java
sdlManager.getFileManager().uploadFile(sdlFile, new CompletionListener() {
    @Override
    public void onComplete(boolean success) {
                            
    }
});
```
#### Uploading multiple files

Sometimes you need to upload more than one file. We've got you covered. Simply create a `List<SdlFile>` object, add your files, and then call:

```java
sdlManager.getFileManager().uploadFiles(sdlFileList, new MultipleFileCompletionListener() {
    @Override
    public void onComplete(Map<String, String> errors) {
                            
    }
});
```
#### Uploading Artwork

As mentioned before, the behavior of `SdlFile` and `SdlArtwork` are the same. But to help separate code, we have also included `uploadArtwork` and `uploadArtworks` methods to the `FileManager` that work the same as their `SdlFile` counterparts shown above.

### File Naming
The file name can only consist of letters (a-Z) and numbers (0-9), otherwise the SDL Core may fail to find the uploaded file (even if it was uploaded successfully).

### File Persistance
`SdlFile` supports uploading persistant images, i.e. images that do not become deleted when your application disconnects. Persistance should be used for images relating to your UI, and not for dynamic aspects, such as Album Artwork.

!!! note
Be aware that persistance will not work if space on the head unit is limited.
!!!

### Overwrite Stored Files
If a file being uploaded has the same name as an already uploaded file, the new file will overwrite the previous file. 

### Check if a File Has Already Been Uploaded
`FileManager` provides 2 methods that allow you to check if a file has been uploaded.

#### getRemoteFileNames()

`getRemoteFileNames()` returns a `List<String>` of the names of files that are uploaded to the head unit. 

```java
List<String> files = sdlManager.getFileManager().getRemoteFileNames();
```

#### hasUploadedFile()

`hasUploadedFile` takes an `SdlFile` and returns a `boolean` of whether it is uploaded or not.

```java
boolean isUploaded = sdlManager.getFileManager().hasUploadedFile(sdlFile);
```


### Check the Amount of File Storage
To find the amount of file storage left on the head unit, use the  the `ListFiles` RPC.

```java
ListFiles listFiles = new ListFiles();
listFiles.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if(response.getSuccess()){
            Integer spaceAvailable = ((ListFilesResponse) response).getSpaceAvailable();
            Log.i("SdlService", "Space available on Core = " + spaceAvailable);
        }else{
            Log.i("SdlService", "Failed to request list of uploaded files.");
        }
    }
});
try{
	sdlManager.sendRPC(listFiles);
} catch (SdlException e) {
	e.printStackTrace();
}
```

### Delete Stored Files

As with uploading, there are two methods that allow you to delete remote files. 

#### For a single file:

To delete a single file, pass in the file name as a string. You can optionally pass in a `CompletionListener`. 

```java
sdlManager.getFileManager().deleteRemoteFileWithName("testFile", new CompletionListener() {
	@Override
	public void onComplete(boolean success) {
				
	}
});
```

#### Multiple files:

To delete multiple files, pass in a list with the names of the files you want to delete. You can optionally pass in a `MultipleFileCompletionListener`.

```java
sdlManager.getFileManager().deleteRemoteFilesWithNames(remoteFiles, new MultipleFileCompletionListener() {
	@Override
	public void onComplete(Map<String, String> errors) {
				
	}
});
```

## Image Specifics
### Image File Type
Images may be formatted as PNG, JPEG, or BMP. Check the `DisplayCapabilities` object provided by `sdlManager.getSystemCapabilityManager().getCapability()` to find out what image formats the head unit supports.

### Image Sizes
If an image is uploaded that is larger than the supported size, that image will be scaled down to accomodate.

#### Image Specifications
ImageName 		  	 | Used in RPC				  |	Details 																							  |	Height 		 | Width  | Type
---------------------|----------------------------|-------------------------------------------------------------------------------------------------------|--------------|--------|-------
softButtonImage		 | Show 					  | Will be shown on softbuttons on the base screen														  | 70px         | 70px   | png, jpg, bmp
choiceImage 		 | CreateInteractionChoiceSet | Will be shown in the manual part of an performInteraction either big (ICON_ONLY) or small (LIST_ONLY) | 70px         | 70px   | png, jpg, bmp
choiceSecondaryImage | CreateInteractionChoiceSet | Will be shown on the right side of an entry in (LIST_ONLY) performInteraction						  | 35px 		 | 35px   | png, jpg, bmp
vrHelpItem			 | SetGlobalProperties		  | Will be shown during voice interaction 																  | 35px 		 | 35px   | png, jpg, bmp
menuIcon			 | SetGlobalProperties		  | Will be shown on the “More…” button 																  | 35px 		 | 35px   | png, jpg, bmp
cmdIcon				 | AddCommand				  | Will be shown for commands in the "More…" menu 														  | 35px 		 | 35px   | png, jpg, bmp
appIcon 			 | SetAppIcon				  | Will be shown as Icon in the "Mobile Apps" menu 													  | 70px 		 | 70px   | png, jpg, bmp
graphic 			 | Show 					  | Will be shown on the basescreen as cover art 														  | 185px 		 | 185px  | png, jpg, bmp