# DeleteInteractionChoiceSet
Deletes an existing Choice Set identified by the parameter interactionChoiceSetID. If the specified interactionChoiceSetID is currently in use by an active PerformInteraction, this call to delete the Choice Set will fail returning an IN_USE resultCode.
### HMI Status Requirements ###

HMILevel needs to be FULL, LIMITED, or BACKGROUND.

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
 			<td> A unique ID that identifies the Choice Set (specified in a previous call to CreateInteractionChoiceSet)</td>
                  <td>Y</td>
                  <td>Min Value: 0 <br>Max Value: 2000000000</td>
  			<td>SmartDeviceLink 1.0</td>
  		</tr>
   </table>

#### Response ###

 If a resultCode of "SUCCESS" is returned, the requested choice set has been created and can now be referenced by the application using the value of interactionChoiceSetID provided by the application.

#### Non-default Result Codes: ####

    - SUCCESS
    - INVALID_DATA
    - OUT_OF_MEMORY
    - TOO_MANY_PENDING_REQUESTS
    - APPLICATION_NOT_REGISTERED
    - GENERIC_ERROR
    - REJECTED
    - INVALID_ID
    - IN_USE  

### Related Operations ###
CreateInteractionChoiceSet

PerformInteraction

### Example Function Call ###
```java
DeleteInteractionChoiceSet deleteChoiceSet = new DeleteInteractionChoiceSet(choiceSetId);
proxy.sendRPCRequest(deleteChoiceSet);
```