# DeleteSubMenu #

Deletes a submenu from the Command Menu.

For a discussion of conflict with user-initiated interaction, see AddCommand.

When an app deletes a submenu that has child commands, those child commands are also deleted.
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
     <td>menuID</td>
     <td>Integer</td>
     <td>Unique ID that identifies the SubMenu to be delete</td>
               <td>Y</td>
               <td>Min Value: 0<br>Max Value: 2000000000</td>
   <td>SmartDeviceLink 1.0</td>
   </tr>
</table>

### Response ###

### Non-default Result Codes: ###

* SUCCESS
* INVALID_DATA
* OUT_OF_MEMORY
* TOO_MANY_PENDING_REQUESTS
* APPLICATION_NOT_REGISTERED
* GENERIC_ERROR
* REJECTED
* INVALID_ID
* IN_USE

### Related Operations ###

* AddCommand
* AddSubMenu
* DeleteCommand

### Example Function Call ###

```java
DeleteSubMenu deleteSubMenu = new DeleteSubMenu(menuID);
proxy.sendRPCRequest(deleteSubMenu);
```