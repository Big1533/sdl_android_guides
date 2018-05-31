# CreateInteractionChoiceSet

Creates a Choice Set which can be used in subsequent PerformInteraction Operations.

A Choice Set is a list of commands (menu or voice) that can be used in certain operations to gain input from the user. Multiple Choices with the same id can be added to SDL from one application.

### HMI Status Requirements ###

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
  			<td>interactionChoiceSetID</td>
  			<td>Integer</td>
  			<td>A unique ID that identifies the Choice Set</td>
                  <td>Y</td>
                  <td>Min Value: 0<br>Max Value: 2000000000</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
  		<tr>
			<td>choiceSet</td>
 			<td>Choice[]</td>
  			<td>Array of one or more elements.</td>
                  <td>Y</td>
                  <td>Min Value: 1<br>Max Value: 100</td>
  			<td>SmartDeviceLink 1.0 </td>
  		</tr>
   </table>

   <b>Note:</b><br>Second Utterance issue with `CreateInteractionChoiceSet` RPC. <br> Before a perform interaction
  is sent you MUST wait for the success from the `CreateInteractionChoiceSet` RPC.<br>
     If you do not wait the system may not recognize the first utterance from the user.
     </p>
    <b>Response</b><br>
    <p>
    Indicates that the corresponding request either failed or succeeded. If the response returns with a SUCCESS result code, this means the Choice Set was created.
    </p>
     <b>Non-default Result Codes:</b><br>

    - SUCCESS
    - INVALID_DATA
    - OUT_OF_MEMORY
    - TOO_MANY_PENDING_REQUESTS
    - APPLICATION_NOT_REGISTERED
    - GENERIC_ERROR
    - REJECTED
    - INVALID_ID
    - DUPLICATE_NAME
    - UNSUPPORTED_RESOURCE

### Related Operations ###
```
- DeleteInteractionChoiceSet
- PerformInteraction
```
### Example Function Call ###

```java
RPCMessage req;
req = RPCRequestFactory.buildCreateInteractionChoiceSet(_choiceSet, choiceSetID, autoIncCorrID++);
_sdlProxy.sendRPCRequest(req);
```
