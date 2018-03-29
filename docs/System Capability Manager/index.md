# System Capability Manager

The System Capability Manager is a central location to obtain capabilities about the currently connected module. Specific capabilities will be returned for a number of given keys (e.g. `NAVIGATION`, `VIDEO_STREAMING`). It also alleviates the need to individually cache results that come from the `RegisterAppInterface` response or from the new `SystemCapabilityQuery`.

There are multiple capabilities that can be retrieved:

| Supported Capabilities |
| --------- |
| NAVIGATION |
| PHONE_CALL |
| VIDEO_STREAMING |
| REMOTE_CONTROL |
| HMI |
| DISPLAY |
| AUDIO_PASSTHROUGH |
| BUTTON |
| HMI_ZONE |
| PRESET_BANK |
| SOFTBUTTON |
| SPEECH |
| VOICE_RECOGNITION |

## Querying Capabilities

Any point after receiving the first `OnHMIStatus` notification from the connected module, you can access the `SystemCapability` manager and its data. Your instance of `SdlProxyALM` will have convenience methods that tie into the `SystemCapabilityManager`.

!!! Note
It is important to query capabilities before you use them. Your app may be used on a variety of head units across different manufacturers and software versions. Never assume that a capability exists.
!!!

For example (obtaining the head unit's `NAVIGATION` capability):

```java
    // First you can check to see if the capability is supported on the module
    if (proxy.isCapabilitySupported(SystemCapabilityType.NAVIGATION){
		// Since the module does support this capability we can query it for more information
		proxy.getCapability(SystemCapabilityType.NAVIGATION, new OnSystemCapabilityListener(){

			@Override
			public void onCapabilityRetrieved(Object capability){
				NavigationCapability navCapability = (NavigationCapability) capability;
				// Now it is possible to get details on how this capability 
				// is supported using the navCapability object
			}

			@Override
			public void onError(String info){
				Log.i(TAG, "Capability could not be retrieved: "+ info);
			}
		});
	}
```

The returned capability needs to be casted into the capability type you requested. From there you can determine whether or not the head unit that the app is connected to can utilize a feature or not. 

### Capability Lists

There are currently 3 responses that come back as Lists: 
- `AUDIO_PASSTHROUGH`
- `BUTTON`
- `SOFTBUTTON`
- `SPEECH`
- `HMI_ZONE`
- `VOICE_RECOGNITION`

We've created a method in the `SystemCapabilityManager` to help cast these lists. Below is an example of its usage:


```java
    public void getCapabilities() {

		proxy.getCapability(SystemCapabilityType.BUTTON, new OnSystemCapabilityListener(){

            @Override
            public void onCapabilityRetrieved(Object capability){
                List<ButtonCapabilities> buttonCapabilityList = SystemCapabilityManager.convertToList(capability, ButtonCapabilities.class);

            }

			@Override
			public void onError(String info){
				Log.i(TAG, "Capability could not be retrieved: "+ info);
			}
		});
	}
```

This method prevents the developer from having to suppress a warning as well as creates a safe way to cast the object to a list.

## Asynchronous vs Synchronous Queries 
Some capabilities will be instantly available after the first `OnHMIStatus` notification. These are parsed from the `RegisterAppInterface` response. However, some capabilities MUST be obtained asynchronously and therefore require a callback to be obtained.  If a capability can be retrieved synchronously another method can be used via the `SdlProxyALM` object, `proxy.getCapability(SystemCapabilityType)`.

|Capability | Async required |
| --------- | ----- |
| NAVIGATION | Yes |
| PHONE_CALL |Yes |
| VIDEO_STREAMING |Yes |
| REMOTE_CONTROL |Yes |
| HMI | No |
| DISPLAY | No |
| AUDIO_PASSTHROUGH | No |
| BUTTON | No |
| HMI_ZONE | No |
| PRESET_BANK | No |
| SOFTBUTTON | No |
| SPEECH | No |
| VOICE_RECOGNITION | No |