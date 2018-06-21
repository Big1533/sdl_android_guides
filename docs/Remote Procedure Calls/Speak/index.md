# Speak 
Speaks a phrase over the vehicle’s audio system using SDL's TTS (text-to-speech) engine. The provided text to be spoken can be simply a text phrase, or it can consist of phoneme specifications to direct SDL's TTS engine to speak a "speech-sculpted" phrase.

Receipt of the Response indicates the completion of the Speak operation, regardless of how the Speak operation may have completed (i.e. successfully, interrupted, terminated, etc.).

Requesting a new Speak operation while the application has another Speak operation already in progress (i.e. no corresponding Response for that in-progress Speak operation has been received yet) will terminate the in-progress Speak operation (causing its corresponding Response to be sent by SDL) and begin the requested Speak operation.

Requesting a new Speak operation while the application has an Alert operation already in progress (i.e. no corresponding Response for that in-progress Alert operation has been received yet) will result in the Alert operation being canceled (indicated in the Response to the Request).

Requesting a new Alert operation while the application has a Speak operation already in progress (i.e. no corresponding Response for that in-progress Speak operation has been received yet) will terminate the in-progress Speak operation (causing its corresponding Response to be sent by SDL) and begin the requested Alert operation.

Requesting a new Speak operation while the application has a PerformInteraction operation already in progress (i.e. no corresponding Response for that in-progress PerformInteraction operation has been received yet) will result in the Speak operation request being rejected (indicated in the Response to the Request).

Requesting a PerformInteraction operation while the application has a Speak operation already in progress (i.e. no corresponding Response for that in-progress Speak operation has been received yet) will terminate the in-progress Speak operation (causing its corresponding Response to be sent by SDL) and begin the requested PerformInteraction operation.

**Note:** Total character limit depends on platform. Chunks are limited to 500 characters; however, you can have multiple TTS chunks.  This could vary according to the VCA.  

### HMI Status Requirements ###
HMILevel:FULL, Limited

AudioStreamingState:Any

SystemContext:MAIN, MENU, VR

**Note:** When Alert is issued with MENU in effect, Alert is queued and "played" when MENU interaction is completed (i.e. SystemContext reverts to MAIN). When Alert is issued with VR in effect, Alert is queued and "played" when VR interaction is completed (i.e. SystemContext reverts to MAIN).

Note: When both Alert and Speak are queued during MENU or VR, they are "played" back in the order in which they were queued, with all existing rules for "collisions" still in effect.

### Request ###
<table border="1" rules="all">
  		<tr>
  			<th>Name</th>
  			<th>Type</th>
  			<th>Description</th>
                  <th>Reg.</th>
                <th>Notes</th>
  			<th> Ver. Available</th>
  		</tr>
  		<tr>
  			<td>ttsChunks</td>
  			<td>String</td>
  			<td>An array of 1-100 TTSChunk structs which, taken together, specify the phrase to be spoken.</td>
  			<td>Y</td>
  			<td>The array must have 1-100 elements. <br>The total length of the phrase composed from the ttsChunks provided must be less than 500 characters or the request will be rejected. <br>Each chunk can be no more than 500 characters.</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
   </table>

### Response ###

This Response notifies the application of the completion, interruption, or failure of a Speak Request.

### Non-default Result Codes: ###

    - SUCCESS
    - INVALID_DATA
    - OUT_OF_MEMORY
	- TOO_MANY_PENDING_REQUESTS
	- APPLICATION_NOT_REGISTERED
	- GENERIC_ERROR
	- REJECTED
	- DISALLOWED
	- ABORTED
### Related Operations ###
Alert

### Example Function Call ###
```java
Speak speak = new Speak(ttsChunks);
proxy.sendRPCRequest(speak);
```