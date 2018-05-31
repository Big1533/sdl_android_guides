# RegisterAppInterface
Registers the application's interface with SDL, declaring properties of the registration, including the messaging interface version, the app name, etc. The mobile application must establish its interface registration with SDL before any other interaction with SDL can take place. The registration lasts until it is terminated either by the application calling the UnregisterAppInterface method, or by SDL sending an OnAppInterfaceUnregistered notification, or by loss of the underlying transport connection, or closing of the underlying message transmission protocol RPC session.

Until the application receives its first OnHMIStatus Notification, its HMI Status is assumed to be: `HMILevel=NONE`, `AudioStreamingState=NOT_AUDIBLE`, `SystemContext=MAIN`.

All SDL resources which the application creates or uses (e.g. Choice Sets, Command Menu, etc.) are associated with the application's interface registration. Therefore, when the interface registration ends, the SDL resources associated with the application are disposed of. As a result, even though the application itself may continue to run on its host platform (e.g. mobile device) after the interface registration terminates, the application will not be able to use the SDL HMI without first establishing a new interface registration and re-creating its required SDL resources. That is, SDL resources created by (or on behalf of) an application do not persist beyond the life span of the interface registration.

Resources and settings whose lifespan is tied to the duration of an application's interface registration:
- 	Choice Sets
- Command Menus (built by successive calls to AddCommand)
-	Media clock timer display value
- Media track display value
- Button subscriptions

The autoActivateID is used to grant an application the HMILevel and AudioStreamingState it had when it last disconnected.

**Note:** The autoActivateID parameter, and associated behavior, is currently ignored by SDL.

When first calling this method (i.e. first time within life cycle of mobile app), an autoActivateID should not be included. After successfully registering an interface, an autoActivateID is returned to the mobile application for it to use in subsequent connections. If the connection between SDL and the mobile application is lost, such as the vehicle is turned off while the application is running, the autoActivateID can then be passed in another call to RegisterAppInterface to re-acquire HMILevel=FULL.

If the application intends to stream audio it is important to indicate so via the isMediaApp parameter. When set to true, audio will reliably stream without any configuration required by the user. When not set, audio may stream, depending on what the user might have manually configured as a media source on SDL®.

There is no time limit for how long the autoActivateID is "valid" (i.e. would confer focus and opt-in)

### HMI Status Requirements ###

HMILevel is not defined before registering.

### Request ####
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
  			<td>MsgVersion</td>
  			<td>MsgVersion</td>
  			<td>Declares what version of the SDL interface the application expects to use with SDL</td>
                  <td>Y</td>
                  <td>To be compatible, app msg major version number must be less than or equal to SDL major version number. <br>If msg versions are incompatible, app has 20 seconds to attempt successful RegisterAppInterface (w.r.t. msg version) on underlying protocol session, else will be terminated. <br>Major version number is a compatibility declaration. Minor version number indicates minor functional variations (e.g. features, capabilities, bug fixes) when sent from SDL to app (in RegisterAppInterface response).<br> However, the minor version number sent from the app to SDL (in RegisterAppInterface request) is ignored by SDL.</td>
  			<td>SmartDeviceLink 1.0 </td>
  		</tr>
  		<tr>
  			<td>appName</td>
  			<td>String</td>
  			<td>The mobile application's name. This name is displayed in the SDL Mobile Applications menu. It also serves as the unique identifier of the application for SDL .</td>
                  <td>Y</td>
                  <td>- Must be 1-100 characters in length.<br>- Must consist of following characters: <br>- May not be the same (by case insensitive comparison) as the name or any synonym of any currently-registered application. </td>
  			<td>SmartDeviceLink 1.0 </td>
  		</tr>
  		<tr>
 			<td>ttsName</td>
      			<td>TTSChunk</td>
  			<td>TTS string for VR recognition of the mobile application name. Meant to overcome any failing on speech engine in properly pronouncing / understanding app name.</td>
                  <td>N</td>
                 <td>- Size must be 1-100<br>- Needs to be unique over all applications<br>- May not be empty.<br>- May not start with a new line character.</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>ngnMediaScreenAppName</td>
 			<td>String</td>
			<td>Provides an abbreviated version of the app name (if necessary) that will be displayed on the NGN media screen.</td>
                  <td>N</td>
                                  <td>- Must be 1-5 characters<br>- If not provided, value will be derived from appName truncated to 5 characters.</td>
  			<td>SmartDeviceLink 1.0 </td>
  		</tr>
  		<tr>
  			<td>vrSynonyms</td>
  			<td>String</td>
  			<td>An array of 1-100 elements, each element containing a voice-recognition synonym by which this app can be called when being addressed in the mobile applications menu.</td>
                  <td>N</td>
                  <td>- Each vr synonym is limited to 40 characters, and there can be 1-100 synonyms in array-<br> May not be the same (by case insensitive comparison) as the name or any synonym of any currently-registered application.</td>
  			<td>SmartDeviceLink 1.0 </td>
 		</tr>
  		<tr>
  			<td>isMediaApplication</td>
  			<td>Boolean</td>
			<td>Indicates that the application will be streaming audio to SDL (via A2DP) that is audible outside of the BT media source.</td>
                  <td>Y</td>
                 <td></td>
  			<td>SmartDeviceLink 1.0 </td>
  		</tr>
  		<tr>
			<td>languageDesired</td>
  			<td>Language</td>
  			<td>An enumeration indicating what language the application intends to use for user interaction (Display, TTS and VR).</td>
                  <td>Y</td>
                  <td>- If the language indicated does not match the active language on SDL, the interface registration will be rejected.<br>- If the user changes the SDL language while this interface registration is active, the interface registration will be terminated. </td>
  			<td>SmartDeviceLink 1.0</td>
 		</tr>
 		<tr>
  			<td>hmiDisplayLanguageDesired</td>
  			<td>Language</td>
			<td>An enumeration indicating what language the application intends to use for user interaction ( Display).</td>
                  <td>Y</td>
                  <td></td>
  			<td>SmartDeviceLink 2.0 </td>
  		</tr>
  		<tr>
  			<td>appHMIType</td>
  			<td>AppHMIType</td>
  			<td>List of all applicable app types stating which classifications to be given to the app.e.g. for platforms this will determine which "corner(s)" the app can populate</td>
                  <td>N</td>
                  <td>Array Minsize: 1<br>Array Maxsize: 100</td>
  			<td>SmartDeviceLink 2.0 </td>
  		</tr>
  		<tr>
  			<td>hashID</td>
  			<td>String</td>
  			<td>ID used to uniquely identify current state of all app data that can persist through connection cycles (e.g. ignition cycles).<br>This registered data (commands, submenus, choice sets, etc.) can be reestablished without needing to explicitly reregister each piece.<br>If omitted, then the previous state of an app's commands, etc. will not be restored.<br>When sending hashID, all RegisterAppInterface parameters should still be provided (e.g. ttsName, etc.). </td>
                 <td>N</td>
                  <td>maxlength:100</td>
			<td>SmartDeviceLink 2.3.1 </td>
 		</tr>
  		<tr>
  			<td>deviceInfo</td>
 			<td>DeviceInfo</td>
  			<td>Various information about connecting device.</td>
                 <td>N</td>
                  <td></td>
  			<td>SmartDeviceLink 2.3.1 </td>
  		</tr>
  		<tr>
 			<td>appID</td>
  			<td>String</td>
  			<td>ID used to validate app with policy table entries</td>
                  <td>Y</td>
                 <td>Maxlength: 100</td>
  			<td>SmartDeviceLink 2.0 </td>
  		</tr>
   </table>

### Response ###
The response message contains essential information about the SDL system that the request was sent to. This information can be used by an application to alter functionality depending on which type of SDL system it is running on. For instance, an application can use the displayCapabilities parameter to see whether the SDL HMI offers a touch screen, a two-line display, or a one-line display.

The application could then use that information to make sure that the text written to the screen is readable for the given platform. If a resultCode other than SUCCESS is received, the application will have to send one or more RegisterAppInterface Requests until one has a result code of SUCCESS; otherwise, the application is not permitted to send any other Requests.

### Non-default Result Codes: ###
```xml
- SUCCESS
- INVALID_DATA
- OUT_OF_MEMORY
- TOO_MANY_PENDING_REQUESTS
- GENERIC_ERROR
- DUPLICATE_NAME
- TOO_MANY_APPLICATIONS
- APPLICATION_REGISTERED_ALREADY
- UNSUPPORTED_VERSION
- WRONG_LANGUAGE
- DISALLOWED
```
### Related Operations ###
```java
- UnregisterAppInterface
- OnAppInterfaceUnregistered
```
### Example Function Call ###
```java

RegisterAppInterface req;
req = RPCRequestFactory.buildRegisterAppInterface(_mainInstance.logTag, false, null);
	_sdlProxy.sendRPCRequest(req);
  ```
