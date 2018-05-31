# SetGlobalProperties 

Sets value(s) for the specified global property(ies).

### HMI Status Requirements ###

HMILevel needs to be `FULL`, `LIMITED`, or `BACKGROUND`.

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
  			<td>helpPrompt</td>
  			<td>TTSChunk</td>
  			<td>The help prompt. An array of text chunks of type TTSChunk. See TTSChunk.The array must have at least one item.</td>
                  <td>N</td>
  			<td>Array must have at least one element.<br>Only optional it timeoutPrompt has been specified.<br>minsize:1<br> maxsize: 100</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>timeoutPrompt</td>
  			<td>TTSChunk</td>
  			<td>Array of one or more TTSChunk elements specifying the help prompt used in an interaction started by PTT.</td>
                  <td>N</td>
  			<td>Array must have at least one element<br>Only optional it helpPrompt has been specified<br> minsize: 1<br> maxsize: 100</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>vrHelpTitle</td>
  			<td>string</td>
  			<td>Text, which is shown as title of the VR help screen used in an interaction started by PTT.</td>
                  <td>N</td>
  			<td>If omitted on supported displays, the default SDL help title will be used. <br> If omitted and one or more vrHelp items are provided, the request will be rejected. <br>maxlength: 500</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>vrHelp</td>
  			<td>VrHelep</td>
  			<td>Items listed in the VR help screen used in an interaction started by PTT.</td>
                  <td>N</td>
  			<td>If omitted on supported displays, the default SDL VR help / What Can I Say? screen will be used<br>If the list of VR Help Items contains nonsequential positions (e.g. [1,2,4]), the RPC will be rejected.<br>If omitted and a vrHelpTitle is provided, the request will be rejected.<br>minsize:1<br> maxsize: 100 </td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>menuTitle</td>
  			<td></td>
  			<td>Optional text to label an app menu button (for certain touchscreen platforms).</td>
                  <td>N</td>
  			<td>maxlength: 500</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>menuIcon</td>
  			<td> Image</td>
  			<td>Optional icon to draw on an app menu button (for certain touchscreen platforms).</td>
                  <td>N</td>
  			<td></td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>keyboardProperties</td>
  			<td>KeyboardProperties</td>
  			<td>On-screen keybaord configuration (if available).</td>
                  <td>N</td>
  			<td></td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
   </table>

**Note**   Your application shall send a SetGlobalProperties to establish an advanced help prompt before sending any voice commands.

### Response ###

 Indicates whether the requested Global Properties were successfully set.

### Non-default Result Codes: ###
```xml
- SUCCESS
- INVALID_DATA
- OUT_OF_MEMORY
- TOO_MANY_PENDING_REQUESTS
- APPLICATION_NOT_REGISTERED
- GENERIC_ERROR
- REJECTED  
- DISALLOWED
```
### Related Operations ###

* ResetGlobalProperties
* Example Function Call
```java
RPCMessage req;
req = RPCRequestFactory.buildSetGlobalProperties (helpText, timeoutText, autoIncCorrID++);
_sdlProxy.sendRPCRequest(req);
```
