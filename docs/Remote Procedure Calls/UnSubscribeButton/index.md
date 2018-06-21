# UnSubscribeButton 

Deletes a subscription to button notifications for the specified button. For more information about button subscriptions, see SubscribeButton.

Application can unsubscribe from a button that is currently being pressed (i.e. has not yet been released), but app will not get button event.

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
  			<td>buttonName</td>
  			<td>ButtonName</td>
  			<td>Name of the button to unsubscribe.</td>
                  <td>Y</td>
  			<td></td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
   </table>

#### Response ####

### Non-default Result Codes: ###

	- SUCCESS
	- INVALID_DATA
	- OUT_OF_MEMORY
	- TOO_MANY_PENDING_REQUESTS
	- APPLICATION_NOT_REGISTERED
	- GENERIC_ERROR
	- UNSUPPORTED_RESOURCE
	- IGNORED
	- REJECTED

### Related Operations ###

* SubscribeButton

### Example Function Call ###

```java
UnsubscribeButton unsubscribeButton = new UnsubscribeButton(ButtonName.OK);
proxy.sendRPCRequest(unsubscribeButton);
```