# PerformInteraction

Performs an application-initiated interaction in which the user can select a Choice from among the specified Choice Sets. For instance, an application may use a PerformInteraction to ask a user to say the name of a song to play. The user's response is only valid if it appears in the specified Choice Sets and is recognized by SDL.

### HMI Status Requirements ###

HMILevel needs to be FULL.

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
  			<td>initialText</td>
  			<td>String</td>
  			<td>Displayed when the interaction begins. This text may be overlaid by the "Listening" prompt during the interaction. Text is displayed on first line of multiline display, and is centered. If text does not fit on line, it will be truncated</td>
                  <td>Y</td>
  			<td>maxlength:500</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>initialPrompt</td>
  			<td>TTSChunk</td>
  			<td>An array of one or more TTSChunks that, taken together, specify what is to be spoken to the user at the start of an interaction.</td>
                  <td>Y</td>
  			<td>minsize:1<br>maxsize:100</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>interactionMode</td>
  			<td>InteractionMode</td>
  			<td>Indicates how user selects interaction choice. User can choose either by voice (VR_ONLY), by visual selection from the menu (MANUAL_ONLY), or by either mode (BOTH). </td>
                  <td>Y</td>
  			<td></td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>interactionChoiceSetIDList</td>
  			<td>Integer</td>
  			<td>Array of one or more Choice Set IDs. User can select any choice from any of the specified Choice Sets.</td>
                  <td>Y</td>
  			<td>minsize:0<br>maxsize:100<br>minvalue:0<br>maxvalue:2000000000</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>helpPrompt</td>
  			<td>TTSChunk</td>
  			<td>An array of TTSChunks which, taken together, specify the help phrase to be spoken when the user says "help" during the VR session. <br>If this parameter is omitted, the help prompt will be constructed by SDL from the first vrCommand of each choice of all the Choice Sets specified in the interactionChoiceSetIDList parameter. </td>
                  <td>N</td>
  			<td>minsize:1<br>maxsize:100<br><p><p>The helpPrompt specified in SetGlobalProperties is not used by PerformInteraction.</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>timeoutPrompt</td>
  			<td>TTSChunk</td>
  			<td>An array of TTSChunks which, taken together, specify the phrase to be spoken when the listen times out during the VR session. <br>If this parameter is omitted, the timeout prompt will be the same as the help prompt (see helpPrompt parameter). </td>
                  <td>N</td>
  			<td>The timeoutPrompt specified in SetGlobalProperties is not used by PerformInteraction. <br>minsize:1<br>maxsize:100</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>timeout</td>
  			<td>Integer</td>
  			<td>The amount of time, in milliseconds, SDL will wait for the user to make a choice (VR or Menu). If this time elapses without the user making a choice, the timeoutPrompt will be spoken. After this timeout value has been reached, the interaction will stop and a subsequent interaction will take place after SDL speaks the timeout prompt. If that times out as well, the interaction will end completely. If omitted, the default is 10000ms.</td>
                  <td>N</td>
  			<td>minvalue:5000<br>maxvalue:100000<br>defvalue:10000</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>vrHelp</td>
  			<td>VrHelpItem</td>
  			<td>Ability to send suggested VR Help Items to display on-screen during Perform Interaction If omitted on supported displays, the default SDL generated list of suggested choices will be displayed.</td>
                  <td>N</td>
  			<td>Min = 1<br>Max = 100</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>interactionLayout</td>
  			<td>LayoutMode</td>
  			<td>See LayoutMode</td>
                  <td>N</td>
  			<td></td>
  			<td>SmartDeviceLink 3.0</td>
  		</tr>
   </table>

### Response ###

Indicates that interaction is complete, or was rejected.
An interaction can complete when the user selects a choice, or when the interaction times out (no user selection was made), or when the interaction was interrupted by the user (pressing PTT or Menu button), or by system events (e.g. phone call), etc.
This is sent to the mobile application after the user makes a choice from the PerformInteraction Request; or if the user fails to make a selection and the Request times out. Two additional parameters, cmdID and triggerSource, are provided with this Response. The cmdID parameter represents the ID of the Choice in the InteractionChoices that was selected by the user. The triggerSource parameter indicates where said Choice selection came from (either Menu or VR).

### Non-default Result Codes: ###

	- SUCCESS
	- INVALID_DATA
	- OUT_OF_MEMORY
	- TOO_MANY_PENDING_REQUESTS
	- APPLICATION_NOT_REGISTERED
	- GENERIC_ERROR
	- REJECTED
	- INVALID_ID
	- DUPLICATE_NAME
	- TIMED_OUT
	- ABORTED
	- UNSUPPORTED_RESOURCE
	- WARNINGS

### Related Operations ###

* CreateInteractionChoiceSet
* DeleteInteractionChoiceSet

### Example Function Call ###

```java
PerformInteraction performInteraction = new PerformInteraction(initialTest, interactionMode, interactionChoiceSetIDList);
proxy.sendRPCRequest(performInteraction);
```