# Show

Updates the application's display text area, regardless of whether or not this text area is visible to the user at the time of the request. The application's display text area remains unchanged until updated by subsequent calls to Show.

The content of the application's display text area is visible to the user when the application's HMILevel is FULL or LIMITED, and the SystemContext=MAIN and no Alert is in progress.

The Show operation cannot be used to create an animated scrolling screen. To avoid distracting the driver, Show commands cannot be issued more than once every 4 seconds. Requests made more frequently than this will be rejected.

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
  			<td>mainField1</td>
  			<td>String</td>
  			<td>Text to be displayed in a single-line display, or in the upper display line in a two-line display.</td>
                  <td>N</td>
  			<td>If this parameter is omitted, the text of mainField1 does not change. <br>If this parameter is an empty string, the field will be cleared. <br>Maxlength = 500</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>mainField2</td>
  			<td>String</td>
  			<td>Text to be displayed on the second display line of a two-line display.</td>
                  <td>N</td>
  			<td>If this parameter is omitted, the text of mainField2 does not change. <br> If this parameter is an empty string, the field will be cleared.<br>If provided and the display is a single-line display, the parameter is ignored.<br>Maxlength = 500</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>mainField3</td>
  			<td>String</td>
  			<td>Text to be displayed on the first display line of the second page.</td>
                  <td>N</td>
  			<td>If this parameter is omitted, the text of mainField3 does not change. <br>f this parameter is an empty string, the field will be cleared.<br>If provided and the display is a single-line display, the parameter is ignored.<br> Maxlength = 500</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>mainField4</td>
  			<td>String</td>
  			<td>Text to be displayed on the second display line of the second page.</td>
                  <td>N</td>
  			<td>If this parameter is omitted, the text of mainField4 does not change. <br>If this parameter is an empty string, the field will be cleared.<br>If provided and the display is a single-line display, the parameter is ignored.<br>Maxlength = 500</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>alignment</td>
  			<td>TextAlignment</td>
  			<td>Specifies how mainField1 and mainField2 text should be aligned on display.</td>
                  <td>N</td>
  			<td>Applies only to mainField1 and mainField2 provided on this call, not to what is already showing in display.<br>If this parameter is omitted, text in both mainField1 and mainField2 will be centered. <br>Has no effect with navigation display</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>statusBar</td>
  			<td>String</td>
  			<td>The text is placed in the status bar area.</td>
                  <td>N</td>
  			<td>Note: The status bar only exists on navigation displays<br>If this parameter is omitted, the status bar text will remain unchanged.<br>If this parameter is an empty string, the field will be cleared.<br>If provided and the display has no status bar, this parameter is ignored.<br>Maxlength = 500</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>mediaClock</td>
  			<td>String</td>
  			<td>Text value for MediaClock field.<br> Has to be properly formatted by Mobile App according to SDL capabilities.<br>If this text is set, any automatic media clock updates previously set with SetMediaClockTimer will be stopped.</td>
                  <td>N</td>
  			<td>Must be properly formatted as described in the MediaClockFormat enumeration. <br>If a value of five spaces is provided, this will clear that field on the display (i.e. the media clock timer field will not display anything) <br>Maxlength = 500</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>mediaTrack</td>
  			<td>String</td>
  			<td>Array of one or more TTSChunk elements specifying the help prompt used in an interaction started by PTT.</td>
                  <td>N</td>
  			<td>If parameter is omitted, the track field remains unchanged.<br>If an empty string is provided, the field will be cleared.<br>This field is only valid for media applications on navigation displays.<br>Maxlength = 500</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>graphic</td>
  			<td>Image</td>
  			<td>Image to be shown on supported displays.</td>
                  <td>N</td>
  			<td>If omitted on supported displays, the displayed graphic shall not change.</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>secondaryGraphic</td>
  			<td>Image</td>
  			<td>	Image struct determining whether static or dynamic secondary image to display in app.<br>If omitted on supported displays, the displayed secondary graphic shall not change.</td>
                  <td>N</td>
  			<td> </td>
  			<td>SmartDeviceLink 2.3.2</td>
  		</tr>
  		<tr>
  			<td>softButtons</td>
  			<td>SoftButton</td>
  			<td>Soft buttons as defined by the App</td>
                  <td>N</td>
  			<td>If omitted on supported displays, the currently displayed SoftButton values will not change.<br>Array Minsize: 0<br>Array Maxsize: 8</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>customPresets</td>
  			<td>String</td>
  			<td>Custom presets as defined by the App.</td>
                  <td>N</td>
  			<td>If omitted on supported displays, the presets will be shown as not defined.<br>Minsize: 0<br> Maxsize: 6</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>

   </table>

### Response ###

#### Non-default Result Codes: ###

	- SUCCESS
	- INVALID_DATA
	- OUT_OF_MEMORY
	- TOO_MANY_PENDING_REQUESTS
	- APPLICATION_NOT_REGISTERED
	- GENERIC_ERROR
	- REJECTED
	- DISALLOWED  
	- UNSUPPORTED_RESOURCE
	- ABORTED

### Related Operations ###

* Alert
* SetMediaClockTimer

### Example Function Call ###
```java
Show show = new Show();
show.setMainField1(mainField1);
show.setMainField2(mainField2);
show.setAlignment(alightment);
proxy.sendRPCRequest(show);
```