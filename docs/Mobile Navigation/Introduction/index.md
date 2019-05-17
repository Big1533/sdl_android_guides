## Mobile Navigation

!!! IMPORTANT
This feature is only available on Android apps. Currently, JavaSE (embedded) and JavaEE (cloud) apps don't support that.
!!!

Mobile Navigation allows map partners to bring their applications into the car and display their maps and turn by turn easily for the user. This feature has a different behavior on the head unit than normal applications. The main differences are:

* Navigation Apps don't use base screen templates. Their main view is the video stream sent from the device
* Navigation Apps can send audio via a binary stream. This will attenuate the current audio source and should be used for navigation commands
* Navigation Apps can receive touch events from the video stream

!!! Note
In order to use SDL's Mobile Navigation feature, the app must have a minimum requirement of Android 4.4 (SDK 19). This is due to using Android's provided video encoder.
!!!

### Connecting an app

The basic connection is the similar for all apps. Please follow [Getting Started](Overview) for more information.

The first difference for a navigation app is the `appHMIType` of `NAVIGATION` that has to be set in the creation of the `SdlManager`. Navigation apps are also non-media apps.

The second difference is the requirement to call the `setSdlSecurity(List<Class<? extends SdlSecurityBase>> secList)` method from the `SdlManager.Builder ` if connecting to an implementation of Core that requires secure video & audio streaming. This method requires an array of Security Managers, which will extend the `SdlSecurityBase` class. These security libraries are provided by the OEMs themselves, and will only work for that OEM. There is not a general catch-all security library.

```java
SdlManager.Builder builder = new SdlManager.Builder(this, APP_ID, APP_NAME, listener);

Vector<AppHMIType> hmiTypes = new Vector<AppHMIType>();
hmiTypes.add(AppHMIType.NAVIGATION);
builder.setAppTypes(hmiTypes);

List<? extends SdlSecurityBase> securityManagers = new ArrayList();
securityManagers.add(OEMSecurityManager1.class);
securityManagers.add(OEMSecurityManager1.class);
builder.setSdlSecurity(securityManagers);

MultiplexTransportConfig mtc = new MultiplexTransportConfig(this, APP_ID, MultiplexTransportConfig.FLAG_MULTI_SECURITY_OFF);
mtc.setRequiresHighBandwidth(true);
builder.setTransportType(transport);

sdlManager = builder.build();
sdlManager.start();
```

!!! Note
When compiling, you must make sure to include all possible OEM's security managers that you wish to support.
!!!

After being registered, the app will start receiving callbacks. One important notification is `ON_HMI_STATUS`, which informs the app about the currently visible application on the head unit. Right after registering, the `hmiLevel` will be `NONE` or `BACKGROUND`. Streaming should commence once the `hmiLevel` has been set to `FULL` by the head unit.
