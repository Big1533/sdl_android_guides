
# Integration Basics

## Getting Started on JavaEE

In this guide, we exclusively use IntelliJ. We are going to set-up a bare-bones application so you get started using SDL.

!!! IMPORTANT
The SDL Java library supports Java 7 and above.
!!!
 

## SmartDeviceLink Service

A SmartDeviceLink Java Service should be created to manage the lifecycle of the SDL session. The `SdlService` should build and start an instance of the `SdlManager` which will automatically connect with a headunit when available. This `SdlManager` will handle sending and receiving messages to and from SDL after connected.

Create a new service and name it appropriately, for this guide we are going to call it `SdlService`. 
 
```java
public class SdlService {
    //...
}
```
 

### Implementing SDL Manager

In order to correctly connect to an SDL enabled head unit developers need to implement methods for the proper creation and disposing of an `SdlManager` in our `SdlService`.

!!! NOTE
An instance of SdlManager cannot be reused after it is closed and properly disposed of. Instead, a new instance must be created. Only one instance of SdlManager should be in use at any given time.
!!!

```java
public class SdlService {

    //The manager handles communication between the application and SDL
    private SdlManager sdlManager = null;

    //...

    private void buildSdlManager(BaseTransportConfig transport) {
        
        if (sdlManager == null) {
           
            // The app type to be used
            Vector<AppHMIType> appType = new Vector<>();
            appType.add(AppHMIType.MEDIA);

            // The manager listener helps you know when certain events that pertain to the SDL Manager happen
            SdlManagerListener listener = new SdlManagerListener() {
                
                @Override
                public void onStart() {
                	// After this callback is triggered the SdlManager can be used to interact with the connected SDL session (updating the display, sending RPCs, etc)
                }

                @Override
                public void onDestroy() {
                    SdlService.this.stopSelf();
                }

                @Override
                public void onError(String info, Exception e) {
                }
            };

            // Create App Icon, this is set in the SdlManager builder
            SdlArtwork appIcon = new SdlArtwork(ICON_FILENAME, FileType.GRAPHIC_PNG, R.mipmap.ic_launcher, true);

            // The manager builder sets options for your session
            SdlManager.Builder builder = new SdlManager.Builder(APP_ID, APP_NAME, listener);
            builder.setAppTypes(appType);
            builder.setTransportType(transport);
            builder.setAppIcon(appIcon);
            sdlManager = builder.build();
            sdlManager.start();
        }

}
```

The `stopSelf()` method from the `SdlManagerListener` is called whenever the manager detects some disconnect in the connection, whether initiated by the app, by SDL, or by the device’s connection.

!!! IMPORTANT
The `sdlManager` must be shutdown properly if this class is shutting down in the respective method using the method `sdlManager.dispose()`.
!!!

### Determining SDL Support
You have the ability to determine a minimum SDL protocol and a minimum SDL RPC version that your app supports. We recommend not setting these values until your app is ready for production. The OEMs you support will help you configure the correct `minimumProtocolVersion` and `minimumRPCVersion` during the application review process.

If a head unit is blocked by protocol version, your app icon will never appear on the head unit's screen. If you configure your app to block by RPC version, it will appear and then quickly disappear. So while blocking with `minimumProtocolVersion` is preferable, `minimumRPCVersion` allows you more granular control over which RPCs will be present.


```java
builder.setMinimumProtocolVersion(new Version("3.0.0"));
builder.setMinimumRPCVersion(new Version("4.0.0"));

```

### Listening for RPC notifications and events

We can listen for specific events using `SdlManager`'s `addOnRPCNotificationListener`. These listeners can be added either in the `onStart()` callback of the `SdlManagerListener` or after it has been triggered. The following example shows how to listen for HMI Status notifications. Additional listeners can be added for specific RPCs by using their corresponding `FunctionID` in place of the `ON_HMI_STATUS` in the following example and casting the `RPCNotification` object to the correct type. 

##### Example of a listener for HMI Status:

```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_HMI_STATUS, new OnRPCNotificationListener() {
		@Override
		public void onNotified(RPCNotification notification) {
			OnHMIStatus status = (OnHMIStatus) notification;
			if (status.getHmiLevel() == HMILevel.HMI_FULL && ((OnHMIStatus) notification).getFirstRun()) {
				// first time in HMI Full
			}
		}
	});
```


### Main Class

//TODO
