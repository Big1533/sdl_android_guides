# Updating from 4.6 to 4.7


## Overview

This guide is to help developers get setup with the SDL Android library version 4.7. It is assumed that the developer is already updated to 4.6 of the library. This version includes the addition of the SdlManagers and a re-working of the transports greatly enhancing the use of the `SdlRouterService` and adding the functionality for secondary transports on supporting versions of SDL Core. 

In this guide we will be focusing on the transitioning from the proxy, which implemented `SdlProxyALM` into using the `SdlManager` system, which includes specialized submanagers that you can interact with through the `SdlManager`. We will follow the naming convention of the guides, highlighting the previous way of implementing SDL and showing the new ways of implementing it.






## Integration Basics

Likely the most change will come to your `SdlService` class. There are going to be two main differences with how this class was set up in 4.6 versus 4.7.

### Removal of IProxyListenerALM

Previously, your SdlService had to implement this listener. This often bloated the class due to the need to override all of its functions. The need to do this has been removed in 4.7, which allows you to set only the listeners that you need. 

In 4.6:

```java
public class SdlService extends Service implements IProxyListenerALM {

    // The proxy handles communication between the application and SDL
    private SdlProxyALM proxy = null;
    
    ...
    
    @Override
    public void someListener(){}
    ...    
}
```

In 4.7 the need to implement `IProxyListenerALM` is gone:

```java
public class SdlService extends Service {

	// The SdlManager exposes the APIs needed to communicate between the application and SDL
	private SdlManager sdlManager = null;
	
	...
}
```

When this change occurs, you will need to remove the previously overridden functions. If your app used any of these, it will help to document which ones they were, as you will need to add in the listeners that you need using the `SdlManager`'s `addOnRPCNotificationListener`.

### Creation of SdlManager

As we no longer want to directly instantiate `SdlProxyALM`, we need to instantiate the `SdlManager`. Setting your application's parameters when instantiating the `SdlManager` is done via the `SdlManager.Builder`. It is also worth adding in the `SdlManagerListener` so that we can receive lifecycle notifcations. What you will have will look like the following:

```java
public class SdlService extends Service {

    //The manager handles communication between the application and SDL
    private SdlManager sdlManager = null;

    //...

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        
        if (sdlManager == null) {
            MultiplexTransportConfig transport = new MultiplexTransportConfig(this, APP_ID, MultiplexTransportConfig.FLAG_MULTI_SECURITY_OFF);
           
            // The app type to be used
            Vector<AppHMIType> appType = new Vector<>();
            appType.add(AppHMIType.MEDIA);

            // The manager listener helps you know when certain events that pertain to the SDL Manager happen
            SdlManagerListener listener = new SdlManagerListener() {
                
                @Override
                public void onStart() {
                	// RPC listeners and other functionality can be called once this callback is triggered.
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
            SdlManager.Builder builder = new SdlManager.Builder(this, APP_ID, APP_NAME, listener);
            builder.setAppTypes(appType);
            builder.setTransportType(transport);
            builder.setAppIcon(appIcon);
            sdlManager = builder.build();
            sdlManager.start();
        }

    //...

}
```

Once you receive the `onStart` callback from `SdlManager`, you can add in your listeners and start adding UI elements. There will be more about adding the UI elements later. The last example in this section will be about adding specific listeners. Because we removed the `IProxyListenerALM` implementation, you will have to set listeners for the needs of your app.

The `SdlManager` also now handles lock screen logic for you. By default, a default lock screen is enabled. A more in depth description of these changes will be described below including customization and disabling of it. Because this implements an activity, we must define it in our `AndroidManifest.xml` file:

```xml
<!-- Required to use the lock screen -->
        <activity android:name="com.smartdevicelink.managers.lockscreen.SDLLockScreenActivity"
                  android:launchMode="singleTop"/>
```

### Listening for events

We can listen for specific events using `SdlManager`'s `addOnRPCNotificationListener`. These listeners can be placed in the `onStart()` callback of the `SdlManagerListener`. We give an example of using this below, to obtain the HMI Status. Other listeners for specific RPCs can be added easily by changing the `FunctionID` to that of the RPC that is to be listened for and by casting the notification to the appropriate type.

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













