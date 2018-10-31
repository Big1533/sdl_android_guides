## Setting the Navigation Destination
Setting a Navigation Destination allows you to send a GPS location that you would like to prompt that user to navigate to using their embedded navigation. When using the `SendLocation` RPC, you will not receive a callback about how the user interacted with this location, only if it was successfully sent to Core and received. It will be handled by Core from that point on using the embedded navigation system.

!!! note
This currently is only supported for Embedded Navigation. This does not work with Mobile Navigation Apps at this time.
!!!

!!! note
`SendLocation` is an RPC that is usually restricted by OEMs. As a result, the OEM you are connecting to may limit app functionality if not approved for usage.
!!!

### Determining the Result of SendLocation
`SendLocation` has 3 possible results that you should expect:

1. SUCCESS - `SendLocation` was successfully sent.
2. INVALID_DATA - The request you sent contains invalid data and was rejected.
3. DISALLOWED - Your app does not have permission to use `SendLocation`.

### Detecting if SendLocation is Available
`SendLocation` is a newer RPC, so there is a possibility that not all head units will support it, especially if you are connected to a head unit that does not have an embedded navigation. To see if `SendLocation` is supported, you may look at `HmiCapabilities` that can be retrieved using `SystemCapabilityManager`.

```java
HMICapabilities hmiCapabilities = (HMICapabilities) sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.HMI);
if (hmiCapabilities.isNavigationAvailable()){
    // SendLocation supported
}else{
    // SendLocation is not supported
}
```

### Using SendLocation
To use `SendLocation`, you must at least include the Longitude and Latitude of the location. You can also include an address, name, description, phone number, and image.

```java
SendLocation sendLocation = new SendLocation();
sendLocation.setLatitudeDegrees(42.877737);
sendLocation.setLongitudeDegrees(-97.380967);
sendLocation.setLocationName("The Center");
sendLocation.setLocationDescription("Center of the United States");

// Create Address
OasisAddress address = new OasisAddress();
address.setSubThoroughfare("900");
address.setThoroughfare("Whiting Dr");
address.setLocality("Yankton");
address.setAdministrativeArea("SD");
address.setPostalCode("57078");
address.setCountryCode("US-SD");
address.setCountryName("United States");

sendLocation.setAddress(address);

// Monitor response
sendLocation.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        Result result = response.getResultCode();
        if(result.equals(Result.SUCCESS)){
            // SendLocation was successfully sent.
        }else if(result.equals(Result.INVALID_DATA)){
            // The request you sent contains invalid data and was rejected.
        }else if(result.equals(Result.DISALLOWED)){
            // Your app does not have permission to use SendLocation.
        }
    }
});

sdlManager.sendRPC(sendLocation);
```
