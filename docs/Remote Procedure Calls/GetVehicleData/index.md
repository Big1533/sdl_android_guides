# GetVehicleData

Non-periodic vehicle data read request.

### HMI Status Requirements ###

HMILevel needs to be `FULL`, `LIMITED`, or `BACKGROUND`.

### Request ###

 <table border="1" rules="all">
  		<tr>
  			<th>Name</th>
  			<th>Type</th>
  			<th>Description</th>
                  <th>Reg.</th>
                <th>Notes</th>
  			<th>Version</th>
 		</tr>
  		<tr>
  			<td>gps</td>
  			<td>Boolean</td>
  			<td>GPS data. See {@linkplain GPS data }for details</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0 </td>
  		</tr>
  		<tr>
  			<td>speed</td>
  			<td>Boolean</td>
  			<td>The vehicle speed in kilometers per hour</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>rpm</td>
  			<td>Boolean</td>
  			<td>The number of revolutions per minute of the engine</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>fuelLevel</td>
  			<td>Boolean</td>
  			<td>The fuel level in the tank (percentage)</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>fuelLevel_State</td>
  			<td>Boolean</td>
  			<td>The fuel level state</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>instantFuelConsumption</td>
  			<td>Boolean</td>
  			<td>The instantaneous fuel consumption in microlitres</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>externalTemperature</td>
  			<td>Boolean</td>
  			<td>The external temperature in degrees celsius</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>vin</td>
  			<td>Boolean</td>
  			<td>Vehicle identification number</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>prndl</td>
  			<td>Boolean</td>
  			<td>Currently selected gear.</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>tirePressure</td>
  			<td>Boolean</td>
  			<td>Tire pressure status</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>odometer</td>
  			<td>Boolean</td>
  			<td>Odometer in km</td>
                  <td>N</td>
                  <td>Max Length: 500</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>beltStatus</td>
  			<td>Boolean</td>
  			<td>The status of the seat belts</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>bodyInformation</td>
  			<td>Boolean</td>
  			<td>The body information including ignition status and internal temp</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0 </td>
  		</tr>
  		<tr>
  			<td>deviceStatus</td>
  			<td>Boolean</td>
  			<td>The device status including signal and battery strength</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>driverBraking</td>
  			<td>Boolean</td>
  			<td>The status of the brake pedal</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>wiperStatus</td>
  			<td>Boolean</td>
  			<td>The status of the wipers</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>headLampStatus</td>
  			<td>Boolean</td>
  			<td>Status of the head lamps</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>engineTorque</td>
  			<td>Boolean</td>
  			<td>Torque value for engine (in Nm) on non-diesel variants</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>accPedalPosition</td>
  			<td>Boolean</td>
  			<td>Accelerator pedal position (percentage depressed)</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>steeringWheelAngle</td>
  			<td>Boolean</td>
  			<td>Current angle of the steering wheel (in deg)</td>
                  <td>N</td>
                  <td>Subscribable</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
   </table>

### Response ###

### Non-default Result Codes: ###
```xml
- SUCCESS
- INVALID_DATA
- OUT_OF_MEMORY
- TOO_MANY_PENDING_REQUESTS
- APPLICATION_NOT_REGISTERED
- GENERIC_ERROR
- REJECTED
- VEHICLE_DATA_NOT_ALLOWED
- VEHICLE_DATA_NOT_AVAILABLE
- USER_DISALLOWED
```
