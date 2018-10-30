# Handling a Language Change 


When a user changes the language on a head unit, an `OnLanguageChange` notification will be sent from Core. Then your app will will disconnect. In order for your app to automatically reconnect to the head unit, there are a few changes to make in the following files: 

* Local SDL Broadcast Receiver
* Local SDL Service

## SDL Broadcast Receiver

When the SDL Service's connection to core is closed, we want to tell our local SDL Broadcast Receiver to restart the SDL Service. To do this, first add a public String in your app's local SDL Broadcast Receiver class that can be included as an extra in a broadcast intent.

`public static final String RECONNECT_LANG_CHANGE = "RECONNECT_LANG_CHANGE";`

Then, override the `onReceive()` method of the local SDL Broadcast Receiver to call `onSdlEnabled()` when receiving that action:

```java
@Override
public void onReceive(Context context, Intent intent) {
	super.onReceive(context, intent); // Required if overriding this method
	
	if (intent != null) {
		String action = intent.getAction();
		if (action != null){
			if(action.equalsIgnoreCase(TransportConstants.START_ROUTER_SERVICE_ACTION)) {
				if (intent.getBooleanExtra(RECONNECT_LANG_CHANGE, false)) {
					onSdlEnabled(context, intent);
				}
			}
		}
	}
}
```

!!! MUST
Be sure to call `super.onReceive(context, intent);` at the start of the method!
!!!

!!! NOTE
This guide also assumes your local SDL Broadcast Receiver implements the `onSdlEnabled()` method as follows:
!!!

```java
@Override
public void onSdlEnabled(Context context, Intent intent) {
	intent.setClass(context, SdlService.class);
	context.startService(intent);
}
```

## SDL Service

We want to tell our local SDL Broadcast Receiver to restart the service when an `OnLanguageChange` notification is received from Core . To do so, add a notification listener as follows: 


```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_LANGUAGE_CHANGE, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        SdlService.this.stopSelf();
        Intent intent = new Intent(TransportConstants.START_ROUTER_SERVICE_ACTION);
        intent.putExtra(SdlReceiver.RECONNECT_LANG_CHANGE, true);
        AndroidTools.sendExplicitBroadcast(context, intent, null);
    }
});
```