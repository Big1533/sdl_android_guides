# ReadDID

Non-periodic vehicle data read request. This is an RPC to get diagnostics data from certain vehicle modules. DIDs of a certain module might differ from vehicle type to vehicle type.

### HMI Status Requirements ###

HMILevel needs to be FULL, LIMITED, or BACKGROUND.

### Request ###

<table border="1" rules="all">
  		<tr>
  			<th>Param Name</th>
  			<th>Type</th>
  			<th>Description</th>
                  <th> Req.</th>
  			<th>Notes</th>
  			<th>Version Available</th>
  		</tr>
  		<tr>
  			<td>ecuName</td>
  			<td>Integer</td>
  			<td>Name of ECU.</td>
                  <td>Y</td>
  			<td>Minvalue: 0<br>Maxvalue: 65535</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>didLocation</td>
  			<td>Integer</td>
  			<td>Get raw data from vehicle data DID location(s).</td>
                  <td>Y</td>
  			<td>Minvalue: 0<br>Maxvalue: 65535</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  			<td>appID</td>
  			<td>Integer</td>
  			<td>ID of the application that requested this RPC.</td>
                  <td>Y</td>
  			<td></td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>

   </table>


### Response ###

### Non-default Result Codes: ###
```
- SUCCESS
- INVALID_DATA
- OUT_OF_MEMORY
- TOO_MANY_PENDING_REQUESTS
- APPLICATION_NOT_REGISTERED
- GENERIC_ERROR
- REJECTED  
- DISALLOWED
- USER_DISALLOWED
- TRUNCATED_DATA
```
