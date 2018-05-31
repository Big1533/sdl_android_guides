# SetMediaClockTimer

Sets the media clock/timer value and the update method (e.g. count-up, countdown, etc.).
### HMI Status Requirements

HMILevel needs to be `FULL`, `LIMITED`, or `BACKGROUND`.

### Request
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
  			<td>startTime</td>
  			<td>StartTime</td>
  			<td>StartTime struct specifying hour, minute, second values to which media clock timer is set.</td>
                  <td>N</td>
  			<td> </td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>endTime</td>
  			<td>StartTime</td>
  			<td> EndTime can be provided for "COUNTUP" and "COUNTDOWN"; to be used to calculate any visual progress bar (if not provided, this feature is ignored)
      	If endTime is greater then startTime for COUNTDOWN or less than startTime for COUNTUP, then the request will return an INVALID_DATA.
      	endTime will be ignored for "RESUME", and "CLEAR"
      	endTime can be sent for "PAUSE", in which case it will update the paused endTime</td>
                  <td>N</td>
  			<td>Array must have at least one element<br>Only optional it helpPrompt has been specified<br> minsize: 1<br> maxsize: 100</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
  			<td>updateMode</td>
  			<td>UpdateMode</td>
  			<td>Specifies how the media clock/timer is to be updated (COUNTUP/COUNTDOWN/PAUSE/RESUME), based at the startTime.</td>
                  <td>Y</td>
  			<td>If "updateMode" is COUNTUP or COUNTDOWN, this parameter must be provided. <br>Will be ignored for PAUSE,RESUME and CLEAR</td>
  			<td>SmartDeviceLink 1.0</td>
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
- REJECTED
- IGNORED
```
### Example Function Calls
```java
RPCMessage req;
req = RPCRequestFactory.buildSetMediaClockTimer (0,15,30, updateMode, autoIncCorrID++);
	_sdlProxy.sendRPCRequest(req);
```
