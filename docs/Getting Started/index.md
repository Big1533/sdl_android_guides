
# Installation

#### Source

Download the latest release from [GitHub](https://github.com/smartdevicelink/sdl_android/releases)

Extract the source code from the archive

Import the code in the top level `sdl_android` folder as a module in Android Studio:

1. Click `File` -> `New` -> `Import Module...` -> Choose the location of `sdl_android` as your Source Directory
2. The module name will automatically be set to `sdl_android`, change this as desired. Click `Next` and then `Finish`

The `sdl_android` library is now a module in your Android Studio project, but it needs to be added as a dependency to your application. Add the following to the gradle dependencies of your project:

```gradle
dependencies {
    compile project(path: ':sdl_android')
}

```

#### Gradle

To compile with the a release of SDL Android, include the following in your app's `build.gradle` file,

```
repositories {
    jcenter()
}
dependencies {
    compile 'com.smartdevicelink:sdl_android:4.+'
}
```

The list of releases can be found [here](https://github.com/smartdevicelink/sdl_android/releases). 

!!! NOTE
For more information, see the _Compiling With Gradle_ guide.
!!!

# Getting Started on Android

In this guide, we exclusively use Android Studio. We are going to set-up a bare-bones application so you get started using SDL.

!!! IMPORTANT
The SDL Mobile library for supports [Android 2.2.x (API Level 8)](https://developer.android.com/about/versions/android-2.2.html) or higher.
!!!
 
## Required System Permissions

In the AndroidManifest for our sample project we need to ensure we have the following system permissions: 

* [Internet](https://developer.android.com/reference/android/Manifest.permission.html#INTERNET) - Used by the mobile library to communicate with a SDL Server
* [Bluetooth](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH) - Primary transport for SDL communication between the device and the vehicle's head-unit
* [Access Network State](https://developer.android.com/reference/android/Manifest.permission.html#ACCESS_NETWORK_STATE) - Required to check if WiFi is enabled on the device

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.company.mySdlApplication">
    
    ...

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.BLUETOOTH"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    ...

</manifest>
```

## SmartDeviceLink Service

A SmartDeviceLink Android Service should be created to manage the lifecycle of an SDL Proxy. The SDL Service enables auto-start by creating the SDL Proxy, which then waits for a connection from SDL. This file also sends and receives messages to and from SDL after connected.

Create a new service and name it appropriately, for this guide we are going to call it `SdlService`. This service must implement the `IProxyListenerALM` interface:
 
```java
public class SdlService extends Service implements IProxyListenerALM {
    // Inherited methods from IProxyListenerALM
}
```
 The IProxyListenerALM interface has most of the callbacks you will receive during the SDL proxy's lifecycle. This includes callbacks for RPC responses.
 
If you created the service using the Android Studio template then the service should have been added to your `AndroidManifest.xml` otherwise the service needs to be defined in the manifest:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.company.mySdlApplication">

    <application>
    
    ...

        <service
        android:name=".SdlService"
        android:enabled="true"/>
    
    </application>

    ...

</manifest>
```

### Entering the Foreground

Because of Android Oreo's requirements, it is mandatory that services enter the foreground for long running tasks. The first bit of integration is ensuring that happens in the `onCreate` method of the `SdlService` or similar. Within the service that implements the SDL lifecycle you will need to add a call to start the service in the foreground. This will include creating a notification to sit in the status bar tray. This information and icons should be relevant for what the service is doing/going to do. If you already start your service in the foreground, you can ignore this section.

```java

public void onCreate() {
    super.onCreate();
    ...
    if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    	NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
     	notificationManager.createNotificationChannel(...);
     	Notification serviceNotification = new Notification.Builder(this, *Notification Channel*)
         	.setContentTitle(...)
         	.setSmallIcon(....)
         	.setLargeIcon(...)
         	.setContentText(...)
         	.setChannelId(channel.getId())
         	.build();
     	startForeground(id, serviceNotification);
     }
}

```
!!! NOTE
The sample code checks if the OS is of Android Oreo or newer to start a foreground service. It is up to the app developer if they wish to start the notification in previous versions.  
!!!

### Exiting the Foreground

It's important that you don't leave you notification in the notification tray as it is very confusing to users. So in the `onDestroy` method in your service, simply call the `stopForeground` method.

```java
@Override
public void onDestroy(){
    //...
	if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.O){
		NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
		if(notificationManager!=null){ //If this is the only notification on your channel
			notificationManager.deleteNotificationChannel(* Notification Channel*);
		}
		stopForeground(true);
	}
}
```

### Implementing SDL Proxy Lifecycle

In order to correctly use a proxy developers need to implement methods for the proper creation and disposing of an SDL Proxy in our SDLService.

!!! NOTE
An instance of SdlProxy cannot be reused after it is closed and properly disposed of. Instead, a new instance must be created. Only one instance of SdlProxy should be in use at any given time.
!!!

```java
public class SdlService extends Service implements IProxyListenerALM {

    //The proxy handles communication between the application and SDL
    private SdlProxyALM proxy = null;

    //...

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        boolean forceConnect = intent !=null && intent.getBooleanExtra(TransportConstants.FORCE_TRANSPORT_CONNECTED, false);
        if (proxy == null) {
            try {
                //Create a new proxy using Bluetooth transport
                //The listener, app name, 
                //whether or not it is a media app and the applicationId are supplied.
                proxy = new SdlProxyALM(this.getBaseContext(),this, "Hello SDL App", true, "8675309");
            } catch (SdlException e) {
                //There was an error creating the proxy
                if (proxy == null) {
                //Stop the SdlService
                    stopSelf();
                }
            }
        }else if(forceConnect){
			proxy.forceOnConnected();
		}

        //use START_STICKY because we want the SDLService to be explicitly started and stopped as needed.
        return START_STICKY;
    }
    
    @Override
    public void onDestroy() {
        //Dispose of the proxy
        if (proxy != null) {
            try {
                proxy.dispose();
            } catch (SdlException e) {
                e.printStackTrace();
            } finally {
                proxy = null;
            }
        }
        
        super.onDestroy();
    }
    
    @Override
    public void onProxyClosed(String info, Exception e, SdlDisconnectedReason reason) {
        //Stop the service
        stopSelf();
    }
    
    //...

}
```

`onProxyClosed()` is called whenever the proxy detects some disconnect in the connection, whether initiated by the app, by SDL, or by the device’s bluetooth connection. As long as the exception does not equal Sdl_PROXY_CYCLED or BLUETOOTH_DISABLED, the proxy would be reset for the exception SDL_PROXY_DISPOSED.

!!! IMPORTANT
We must properly dispose of our proxy in the `onDestroy()` method because SDL will issue an error that it lost connection with the app if the connection fails before calling `proxy.dispose()`.
!!!

### Implementing IProxyListenerALM Methods

#### onOnHMIStatus()

In our `SdlService`, the `onOnHMIStatus()` method is where you should control your application with SDL various HMI Statuses. When you receive the first HMI_FULL, you should initialize your app on SDL by subscribing to buttons, registering addcommands, sending an initial show or speak command, etc.

```java
@Override
public void onOnHMIStatus(OnHMIStatus notification) {

    switch(notification.getHmiLevel()) {
        case HMI_FULL:
            //send welcome message, addcommands, subscribe to buttons ect
            break;
        case HMI_LIMITED:
            break;
        case HMI_BACKGROUND:
            break;
        case HMI_NONE:
            break;
        default:
            return;
    }
}
```

## SmartDeviceLink Router Service

The `SdlRouterService` will listen for a bluetooth connection with an SDL enabled module. When a connection happens, it will alert all SDL enabled apps that a connection has been established and they should start their SDL services.

We must implement a local copy of the `SdlRouterService` into our project. The class doesn't need any modification, it's just important that we include it. We will extend the `com.smartdevicelink.transport.SdlRouterService` in our class named `SdlRouterService`:

!!! NOTE
Do not include an import for `com.smartdevicelink.transport.SdlRouterService`. Otherwise, we will get an error for `'SdlRouterService' is already defined in this compilation unit`.
!!!

```Java
public class SdlRouterService extends  com.smartdevicelink.transport.SdlRouterService {
//Nothing to do here
}
```
!!! MUST
The local extension of the `com.smartdevicelink.transport.SdlRouterService` must be named `SdlRouterService`. 
!!!

!!! MUST
Make sure this local class (SdlRouterService.java) is in the same package of SdlReceiver.java (described below)
!!!

If you created the service using the Android Studio template then the service should have been added to your `AndroidManifest.xml` otherwise the service needs to be added in the manifest. Because we want our service to be seen by other SDL enabled apps, we need to set `android:exported="true"`. The system may issue a lint warning because of this, so we can suppress that using `tools:ignore="ExportedService"`.  Once added, it should be defined like below:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.company.mySdlApplication">

    <application>
    
    ...

        <service
        	android:name="com.company.mySdlApplication.SdlRouterService"
        	android:exported="true" 
        	android:process="com.smartdevicelink.router"
            tools:ignore="ExportedService">
            <intent-filter>
                <action android:name="com.smartdevicelink.router.service"/>
            </intent-filter>
            <meta-data android:name="@string/sdl_router_service_version_name"  android:value="@integer/sdl_router_service_version_value" />
        </service>
    
    </application>

    ...

</manifest>
```

!!! MUST
The `SdlRouterService` must be placed in a separate process with the name `com.smartdevicelink.router`. If it is not in that process during it's start up it will stop itself.
!!!

!!! NOTE
Setting `android:name` to `@string/sdl_router_service_version_name` for the router service metadata may cause issues with some app packaging and analyzing tools like aapt. You can avoid that by hardcoding the string value instead of using a string reference.
!!!

```xml
<meta-data android:name="sdl_router_version"  android:value="@integer/sdl_router_service_version_value" />
```

### Intent Filter

```xml
<intent-filter>
    <action android:name="com.smartdevicelink.router.service"/>
</intent-filter>
```

The new versions of the SDL Android library rely on the `com.smartdevicelink.router.service` action to query SDL enabled apps that host router services. This allows the library to determine which router service to start.

!!! MUST
This `intent-filter` MUST be included.
!!!

### Metadata

#### Router Service Version

```xml
<meta-data android:name="@string/sdl_router_service_version_name"  android:value="@integer/sdl_router_service_version_value" />
```

Adding the `sdl_router_service_version` metadata allows the library to know the version of the router service that the app is using. This makes it simpler for the library to choose the newest router service when multiple router services are available.

#### Custom Router Service

```xml
<meta-data android:name="@string/sdl_router_service_is_custom_name" android:value="false" />
```

!!! NOTE
This is only for specific OEM applications, therefore normal developers do not need to worry about this.
!!!

Some OEMs choose to implement custom router services. Setting the `sdl_router_service_is_custom_name` metadata value to `true` means that the app is using something custom over the default router service that is included in the SDL Android library. Do not include this `meta-data` entry unless you know what you are doing. 


## SmartDeviceLink Broadcast Receiver

The Android implementation of the SdlProxy relies heavily on the OS's bluetooth intents. When the phone is connected to SDL, the app needs to create a SdlProxy, which publishes an SDP record for SDL to connect to. As mentioned previously, a SdlProxy cannot be re-used. When a disconnect between the app and SDL occurs, the current proxy must be disposed of and a new one created.

The SDL Android library has a custom broadcast receiver named `SdlBroadcastReceiver` that should be used as the base for your BroadcastReceiver. It is a child class of Android's BroadcastReceiver so all normal flow and attributes will be available. Two abstract methods will be automatically populate the class, we will fill them out soon.

Create a new SdlBroadcastReceiver and name it appropriately, for this guide we are going to call it `SdlReceiver`:

```java
public class SdlReceiver extends SdlBroadcastReceiver {

      	@Override
	public void onSdlEnabled(Context context, Intent intent) {
		//...
		
	}


	@Override
	public Class<? extends SdlRouterService> defineLocalSdlRouterClass() {
        //...
	}
}
```

!!! MUST 
SdlBroadcastReceiver must call super if ```onReceive``` is overridden
!!!

``` java
	@Override
	public void onReceive(Context context, Intent intent) {
		super.onReceive(context, intent);
		//your code here
	}
```

If you created the BroadcastReceiver using the Android Studio template then the service should have been added to your `AndroidManifest.xml` otherwise the receiver needs to be defined in the manifest. Regardless, the manifest needs to be edited so that the `SdlBroadcastReceiver` needs to respond to the following intents:

* [android.bluetooth.device.action.ACL_CONNECTED](https://developer.android.com/reference/android/bluetooth/BluetoothDevice.html#ACTION_ACL_CONNECTED)
* [android.bluetooth.device.action.ACL_DISCONNECTED](https://developer.android.com/reference/android/bluetooth/BluetoothDevice.html#ACTION_ACL_DISCONNECTED)
* [android.bluetooth.adapter.action.STATE_CHANGED](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html#ACTION_CONNECTION_STATE_CHANGED)
* [android.media.AUDIO_BECOMING_NOISY](https://developer.android.com/reference/android/media/AudioManager.html#ACTION_AUDIO_BECOMING_NOISY)
* sdl.router.startservice

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.company.mySdlApplication">

    <application>

        ...

        <receiver
            android:name=".SdlReceiver"
            android:exported="true"
            android:enabled="true">
    
            <intent-filter>
                <action android:name="android.bluetooth.device.action.ACL_CONNECTED" />
                <action android:name="android.bluetooth.device.action.ACL_DISCONNECTED"/>
                <action android:name="android.bluetooth.adapter.action.STATE_CHANGED"/>
                <action android:name="android.media.AUDIO_BECOMING_NOISY" />
                <action android:name="sdl.router.startservice" />
            </intent-filter>
    
        </receiver>
    
    </application>

...

</manifest>
```
!!! NOTE
The intent `sdl.router.startservice` is a custom intent that will come from the SdlRouterService to tell us that we have just connected to an SDL enabled piece of hardware.
!!!

!!! MUST
SdlBroadcastReceiver has to be exported, or it will not work correctly
!!!

Next, we want to make sure we supply our instance of the SdlBroadcastService with our local copy of the SdlRouterService. We do this by simply returning the class object in the method defineLocalSdlRouterClass:

```java
public class SdlReceiver extends SdlBroadcastReceiver {
          	@Override
	public void onSdlEnabled(Context context, Intent intent) {
		//..
	}

	@Override
	public Class<? extends SdlRouterService> defineLocalSdlRouterClass() {
        //Return a local copy of the SdlRouterService located in your project
		return com.company.mySdlApplication.SdlRouterService.class;
	}
}
```


We want to start the SDL Proxy when an SDL connection is made via the `SdlRouterService`. We do this by taking action in the onSdlEnabled method:

!!! MUST
Apps must start their service in the foreground as of Android Oreo (API 26).
!!!

```java
public class SdlReceiver extends SdlBroadcastReceiver {
   
   @Override
	public void onSdlEnabled(Context context, Intent intent) {
		//Use the provided intent but set the class to the SdlService
		intent.setClass(context, SdlService.class);
		if(Build.VERSION.SDK_INT < Build.VERSION_CODES.O) {
			context.startService(intent);
		}else{
			context.startForegroundService(intent);
		}
	}


	@Override
	public Class<? extends SdlRouterService> defineLocalSdlRouterClass() {
        //Return a local copy of the SdlRouterService located in your project
		return com.company.mySdlApplication.SdlRouterService.class;
	}
}
```
!!! IMPORTANT
The `onSdlEnabled` method will be the main start point for our SDL connection session. We define exactly what we want to happen when we find out we are connected to SDL enabled hardware.
!!!

### Main Activity

Now that the basic connection infrastructure is in place, we should add methods to start the SdlService when our application starts. In `onCreate()` in your main activity, you need to call a method that will check to see if there is currently an SDL connection made. If there is one, the onSdlEnabled method will be called and we will follow the flow we already set up. In our `MainActivity.java` we need to check for an SDL connection:

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //If we are connected to a module we want to start our SdlService
		SdlReceiver.queryForConnectedService(this);
    }
}
```

