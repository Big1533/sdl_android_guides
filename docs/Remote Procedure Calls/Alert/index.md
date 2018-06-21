# Alert

Provides information to the user using either TTS, the Display or both and can include a system-generated alert tone.

  - The displayed portion of the Alert, if any, will persist until the specified timeout has elapsed, or the Alert is preempted.
  - An Alert will preempt (abort) any smartdevicelink Operation that is in-progress, except an already-in-progress Alert.
  - An Alert cannot be preempted by any smartdevicelink Operation.
  - An Alert can be preempted by a user action (button push).
  - An Alert will fail if it is issued while another Alert is in progress. Assuming it's coming from the same app, the current Alert gets ABORTED and the new Alert will get displayed/audible.
 - Although each Alert parameter is optional, in fact each Alert request must supply at least one of the following **parameters:**
 - alertText1
- alertText2
- alertText3
-	ttsChunks

### HMI Status Requirements ###

HMILevel needs to be FULL or LIMITED.

If the app has been granted function group Notification the HMILevel can also be BACKGROUND
### Request ###
<table border="1" rules="all">
  		<tr>
  			<th>Name</th>
 			<th>Type</th>
 			<th>Description</th>
                 <th> Req.</th>
  			<th>Notes</th>
  			<th>Version Available</th>
 		</tr>
 		<tr>
  			<td>alertText1</td>
  			<td>String</td>
 			<td>Text to be displayed in the first field of the display during the Alert. </td>
                  <td>N</td>
 			<td> Length is limited to what is indicated in RegisterAppInterface response. If omitted, top display line will be cleared. Text is always centered</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
 		<tr>
  			<td>alertText2</td>
  			<td>String</td>
  			<td>Text to be displayed in the second field of the display during the Alert. </td>
                  <td>N</td>
  			<td> Only permitted if HMI supports a second display line.<br>	Length is limited to what is indicated in RegisterAppInterface response. <br>	If omitted, second display line will be cleared.  </td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>alertText3</td>
  			<td>String</td>
  			<td>Text to be displayed in the third field of the display during the Alert.</td>
                  <td>N</td>
  			<td>Array must have a least one element. </td>
  			<td>SmartDeviceLink 1.0</td>
		</tr>
  		<tr>
  			<td>ttsChunks</td>
  			<td>TTSChunk[]</td>
  			<td>Array of type TTSChunk which, taken together, specify what is to be spoken to the user.</td>
                  <td>N</td>
  			<td>Array must have a least one element. </td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>duration</td>
  			<td>Integer</td>
  			<td>The duration of the displayed portion of the alert, in milliseconds.<br> After this amount of time has passed, the display fields<br> alertText1 and alertText2 will revert to what was displayed in those fields before the alert began.</td>
                  <td>N</td>
  			<td>Min Value: 3000 <br> Max Value: 10000<br> If omitted, the default is 5000 milliseconds</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>playTone</td>
  			<td>Boolean</td>
  			<td>Specifies whether the alert tone should be played before the TTS (if any) is spoken.</td>
                  <td>N</td>
  			<td>If omitted, default is true.</td>
 			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>softButtons</td>
  			<td>SoftButton[]</td>
  			<td>Specifies the softbuttons, the apps wants to use in this alert.</td>
                  <td></td>
  			<td>If omitted on supported displays, the alert will not have any SoftButton.<br> ArrayMin: 0<br> ArrayMax: 4</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>progressIndicator</td>
  			<td>Boolean</td>
  			<td>If supported on the given platform, the alert GUI will include some sort of animation indicating that loading of a feature is progressing.  e.g. a spinning wheel or hourglass, etc.</td>
                  <td>N</td>
  			<td></td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
   </table>

### Response###

If a resultCode of "SUCCESS" is returned, the request was accepted by SDL. By the time the corresponding response is received, the Alert will have completed.
### Non-default Result Codes: ###
	- SUCCESS
	- INVALID_DATA
	- OUT_OF_MEMORY
	- TOO_MANY_PENDING_REQUESTS
	- APPLICATION_NOT_REGISTERED
	- GENERIC_ERROR  
	- REJECTED
	- ABORTED
	- DISALLOWED
	- USER_DISALLOWED
	- UNSUPPORTED_RESOURCE


   <table border="1" rules="all">
    		<tr>
    			<th>Name</th>
    			<th>Type</th>
    			<th>Description</th>
                    <th> Req.</th>
    			<th>Notes</th>
    			<th>Version Available</th>
    		</tr>
   		<tr>
   			<td>tryAgainTime</td>
    			<td> Integer</td>
    			<td>Amount of time (in seconds) that an app must wait before resending an alert.</td>
                    <td>N</td>
   			<td> If provided, another system event or overlay currently has a higher priority than this alert. An app must not send an alert without waiting at least the amount of time dictated.</td>
    			<td>smartdevicelink 1.0</td>
    		</tr>  
    </table>

### Related Operations ###
Show
Speak

### Example Function Call ###
```java
Alert alert = new Alert();
alert.setAlertText1("alert1");
alert.setDuration(10);
alert.setPlayTone(true);
proxy.sendRPCRequest(alert);
```
