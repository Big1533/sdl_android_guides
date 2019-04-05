## Get Vehicle Data

Use the GetVehicleData RPC request to get vehicle data. The HMI level must be FULL, LIMITED, or BACKGROUND in order to get data.

Each vehicle manufacturer decides which data it will expose. Please check the [PermissionManager](/guides/android/permission-manager/) to find out which data types your app currently has access to for the connected head unit.

!!! note
You may only ask for vehicle data that is available to your appName & appId combination. These will be specified by each OEM separately.
!!!

| Vehicle Data | Parameter Name  |  Description |
| ------------- | ------------- | ------------- |
| GPS | gps | Longitude and latitude, current time in UTC, degree of precision, altitude, heading, speed, satellite data vs dead reckoning, and supported dimensions of the GPS |
| Speed | speed | Speed in KPH |
| RPM | rpm | The number of revolutions per minute of the engine |
| Fuel level | fuelLevel | The fuel level in the tank (percentage) |
| Fuel level state | fuelLevel_State | The fuel level state: unknown, normal, low, fault, alert, or not supported |
| Fuel range | fuelRange |  The estimate range in KM the vehicle can travel based on fuel level and consumption |
| Instant fuel consumption | instantFuelConsumption | The instantaneous fuel consumption in microlitres |
| External temperature | externalTemperature | The external temperature in degrees celsius |
| VIN | vin | The Vehicle Identification Number |
| PRNDL | prndl | The selected gear the car is in: park, reverse, neutral, drive, sport, low gear, first, second, third, fourth, fifth, sixth, seventh or eighth gear, unknown, or fault |
| Tire pressure | tirePressure | Tire status of each wheel in the vehicle: normal, low, fault, alert, or not supported. Warning light status for the tire pressure: off, on, flash, or not used |
| Odometer | odometer | Odometer reading in km |
| Belt status | beltStatus | The status of each of the seat belts: no, yes, not supported, fault, or no event |
| Body information | bodyInformation | Door ajar status for each door. The Ignition status. The ignition stable status. The park brake active status. |
| Device status | deviceStatus | Contains information about the smartphone device. Is voice recognition on or off, has a bluetooth connection been established, is a call active, is the phone in roaming mode, is a text message available, the battery level, the status of the mono and stereo output channels, the signal level, the primary audio source, whether or not an emergency call is currently taking place |
| Driver braking | driverBraking | The status of the brake pedal: yes, no, no event, fault, not supported |
| Wiper status | wiperStatus | The status of the wipers: off, automatic off, off moving, manual interaction off, manual interaction on, manual low, manual high, manual flick, wash, automatic low, automatic high, courtesy wipe, automatic adjust, stalled, no data exists |
| Head lamp status | headLampStatus | Status of the head lamps: whether or not the low and high beams are on or off. The ambient light sensor status: night, twilight 1, twilight 2, twilight 3, twilight 4, day, unknown, invalid |
| Engine torque | engineTorque | Torque value for engine (in Nm) on non-diesel variants |
| Engine oil life | engineOilLife | The estimated percentage of remaining oil life of the engine |
| Acceleration pedal position | accPedalPosition | Accelerator pedal position (percentage depressed) |
| Steering wheel angle | steeringWheelAngle | Current angle of the steering wheel (in degrees) |
| E-Call information | eCallInfo | Information about the status of an emergency call |
| Airbag status | airbagStatus | Status of each of the airbags in the vehicle: yes, no, no event, not supported, fault |
| Emergency event | emergencyEvent | The type of emergency: frontal, side, rear, rollover, no event, not supported, fault. Fuel cutoff status: normal operation, fuel is cut off, fault. The roll over status: yes, no, no event, not supported, fault. The maximum change in velocity. Whether or not multiple emergency events have occurred |
| Cluster mode status | clusterModeStatus | Whether or not the power mode is active. The power mode qualification status: power mode undefined, power mode evaluation in progress, not defined, power mode ok. The car mode status: normal, factory, transport, or crash. The power mode status: key out, key recently out, key approved, post accessory, accessory, post ignition, ignition on, running, crank |
| My key | myKey | Information about whether or not the emergency 911 override has been activated |
| Turn signal | turnSignal | The status of the turn light indicator |
| Electronic park brake status | electronicParkBrakeStatus | The status of the park brake as provided by Electric Park Brake (EPB) system |
| Cloud app vehicle id | cloudAppVehicleID | The id for the vehicle when connecting to cloud applications |

### Single Time Vehicle Data Retrieval
Using `GetVehicleData`, we can ask for vehicle data a single time, if needed. 

```java
GetVehicleData vdRequest = new GetVehicleData();
vdRequest.setPrndl(true);
vdRequest.setOnRPCResponseListener(new OnRPCResponseListener() {
	@Override
	public void onResponse(int correlationId, RPCResponse response) {
		if(response.getSuccess()){
			PRNDL prndl = ((GetVehicleDataResponse) response).getPrndl();
			Log.i("SdlService", "PRNDL status: " + prndl.toString());
		}else{
			Log.i("SdlService", "GetVehicleData was rejected.");
		}
	}
});
sdlManager.sendRPC(vdRequest);
```

### Subscribing to Vehicle Data
Subscribing to vehicle data allows you to get notified whenever we have new data available. This data should not be relied upon being received in a consistent manner. New vehicle data is available roughly every second.


**First**, you should add a notification listener for `OnVehicleData` notification: 

```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_VEHICLE_DATA, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnVehicleData onVehicleDataNotification = (OnVehicleData) notification;
        if (onVehicleDataNotification.getPrndl() != null) {
            Log.i("SdlService", "PRNDL status was updated to: " + onVehicleDataNotification.getPrndl());
        }
    }
});
```


**Then**, send the Subscribe Vehicle Data Request:

```java
SubscribeVehicleData subscribeRequest = new SubscribeVehicleData();
subscribeRequest.setPrndl(true);
subscribeRequest.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if(response.getSuccess()){
            Log.i("SdlService", "Successfully subscribed to vehicle data.");
        }else{
            Log.i("SdlService", "Request to subscribe to vehicle data was rejected.");
        }
    }
}); 
sdlManager.sendRPC(subscribeRequest);
```

**After that**, the `onNotified` method should be called when there is an update to the subscribed vehicle data.

### Unsubscribing from Vehicle Data
Sometimes you may not always need all of the vehicle data you are listening to. We suggest that you only are subscribing when the vehicle data is needed. To stop listening to specific vehicle data items, utilize `UnsubscribeVehicleData`.

```java
UnsubscribeVehicleData unsubscribeRequest = new UnsubscribeVehicleData();
unsubscribeRequest.setPrndl(true); // unsubscribe to PRNDL data
unsubscribeRequest.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if(response.getSuccess()){
            Log.i("SdlService", "Successfully unsubscribed to vehicle data.");
        }else{
            Log.i("SdlService", "Request to unsubscribe to vehicle data was rejected.");
        }
    }
});
sdlManager.sendRPC(unsubscribeRequest);
```
