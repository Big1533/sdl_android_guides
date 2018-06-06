# GetDTCs

This RPC allows you to request diagnostic module trouble codes from a vehicle module.
HMI Status Requirements

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
  			<td>ecuName</td>
  			<td>Integer</td>
  			<td>Name of ECU.</td>
                  <td>Y</td>
                  <td>Min Value: 0<br>Max Value: 65535</td>
  			<td>SmartDeviceLink 2.0 </td>
  		</tr>
  		<tr>
  			<td>dtcMask</td>
  			<td>Integer</td>
  			<td>DTC Mask Byte to be sent in diagnostic request to module.</td>
                  <td>N</td>
                  <td>Min Value: 0<br>Max Value: 255</td>
  			<td>SmartDeviceLink 2.0 </td>
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
