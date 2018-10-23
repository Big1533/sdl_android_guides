## Calling a Phone Number
Dialing a Phone Number allows you to send a phone number to dial on the user's phone. Regardless of platform, you must be sure that a device is connected via Bluetooth for this RPC to work. If it is not connected, you will receive a REJECTED `Result`.

!!! note
`DialNumber` is an RPC that is usually restricted by OEMs. As a result, the OEM you are connecting to may limit app functionality if not approved for usage.
!!!

### Determining the Result of DialNumber
`DialNumber` has 3 possible results that you should expect:

1. SUCCESS - `DialNumber` was successfully sent, and a phone call was initiated by the user.
2. REJECTED - `DialNumber` was sent, and a phone call was cancelled by the user. Also, this could mean that there is no phone connected via Bluetooth.
3. DISALLOWED - Your app does not have permission to use `DialNumber`.

### Detecting if DialNumber is Available
`DialNumber` is a newer RPC, so there is a possibility that not all head units will support it. To see if `DialNumber` is supported, you may look at the `HMICapabilities` that can be retrieved using `SystemCapabilityManager`.

```java
HMICapabilities hmiCapabilities = (HMICapabilities) sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.HMI);
if(hmiCapabilities.isPhoneCallAvailable()){
    // DialNumber supported
}else{
    // DialNumber is not supported
}
```

### How to Use
!!! note
For DialNumber, all characters are stripped except for `0`-`9`, `*`, `#`, `,`, `;`, and `+`
!!!

```java
DialNumber dialNumber = new DialNumber();
dialNumber.setNumber("1238675309");
dialNumber.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        Result result = response.getResultCode();
        if(result.equals(Result.SUCCESS)){
            // `DialNumber` was successfully sent, and a phone call was initiated by the user.
        }else if(result.equals(Result.REJECTED)){
            // `DialNumber` was sent, and a phone call was cancelled by the user. Also, this could mean that there is no phone connected via Bluetooth.
        }else if(result.equals(Result.DISALLOWED)){
            // Your app does not have permission to use DialNumber.
        }
    }
});
    
sdlManager.sendRPC(dialNumber);
```
