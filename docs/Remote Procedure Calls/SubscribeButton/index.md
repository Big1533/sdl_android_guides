# SubscribeButton

Establishes a subscription to button notifications for HMI buttons. Buttons are not necessarily physical buttons, but can also be "soft" buttons on a touch screen, depending on the display in the vehicle. Once subscribed to a particular button, an application will receive both OnButtonEvent and OnButtonPress notifications whenever that button is pressed. The application may also unsubscribe from notifications for a button by invoking the UnsubscribeButton operation.

When a button is depressed, an OnButtonEvent notification is sent to the application with a ButtonEventMode of BUTTONDOWN. When that same button is released, an OnButtonEvent notification is sent to the application with a ButtonEventMode of BUTTONUP.

When the duration of a button depression (that is, time between depression and release) is less than two seconds, an OnButtonPress notification is sent to the application (at the moment the button is released) with a ButtonPressMode of SHORT. When the duration is two or more seconds, an OnButtonPress notification is sent to the application (at the moment the two seconds have elapsed) with a ButtonPressMode of LONG.
The purpose of OnButtonPress notifications is to allow for programmatic detection of long button presses similar to those used to store presets while listening to the radio, for example.

When a button is depressed and released, the sequence in which notifications will be sent to the application is as follows:

For short presses:
* OnButtonEvent (ButtonEventMode = BUTTONDOWN)
* OnButtonEvent (ButtonEventMode = BUTTONUP)
* OnButtonPress (ButtonPressMode = SHORT)

For long presses:
* OnButtonEvent (ButtonEventMode = BUTTONDOWN)
* OnButtonPress (ButtonPressMode = LONG)
* OnButtonEvent (ButtonEventMode = BUTTONUP)

### HMI Status Requirements

HMILevel needs to be `FULL`, `LIMITED`, or `BACKGROUND`.

### Request

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
  			<td>buttonName</td>
  			<td>ButtonName</td>
  			<td>Name of the button to subscribe.</td>
                  <td>Y</td>
                  <td></td>
  			<td>SmartDeviceLink 1.0 </td>
  		</tr>
   </table>

### Response

### Non-default Result Codes:
```xml
- SUCCESS
- INVALID_DATA
- OUT_OF_MEMORY
- TOO_MANY_PENDING_REQUESTS
- APPLICATION_NOT_REGISTERED
- GENERIC_ERROR
- UNSUPPORTED_RESOURCE
- IGNORED
- REJECTED
```

### Related Operations

* UnsubscribeButton

### Example Function Call

```java
SubscribeButton buttonReq;
    	buttonReq = RPCRequestFactory.buildSubscribeButton(ButtonName.OK, autoIncCorrID++);
_sdlProxy.sendRPCRequest(buttonReq);
```
