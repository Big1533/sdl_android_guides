# DeleteCommand #

Removes a command from the Command Menu

For a discussion of conflict with user-initiated interaction, see AddCommand.
### HMI Status Requirements ###

HMILevel needs to be FULL, LIMITED, or BACKGROUND.

### Request ###
<b>Parameter List</b>
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
     <td>cmdID</td>
     <td>Integer</td>
     <td>Unique ID that identifies the Command to be deleted from Command Menu</td>
               <td>Y</td>
               <td>Min Value: 0<br>Max Value: 2000000000</td>
     <td>SmartDeviceLink 1.0</td>
   </tr>
</table>

### Response ###
Indicates that the corresponding request either failed or succeeded. If the response returns with a SUCCESS result code, this means a command was removed from the Command Menu successfully.

### Non-default Result Codes: ###
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

- AddCommand
- AddSubMenu
- DeleteSubMenu

### Example Function Call ###
```java
DeleteCommand deleteCommand = new DeleteCommand(commandID);
proxy.sendRPCRequest(deleteCommand);
```