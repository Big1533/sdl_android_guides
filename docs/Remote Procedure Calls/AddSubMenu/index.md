# AddSubMenu #
Adds a SubMenu to the Command Menu  A SubMenu can only be added to the Top Level Menu (i.e. a SubMenu cannot be added to a SubMenu), and may only contain commands as children.
For a discussion of conflict with user-initiated interaction, see AddCommand.

### HMI Status Requirements ###

HMILevel needs to be FULL, LIMITED, or BACKGROUND.

### Request ###

<b>Parameter List</b>
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
     <td>menuID</td>
     <td>Integer</td>
   <td>Unique ID that identifies this sub menu. This value is used in AddCommand to which SubMenu is the parent of the command being added.</td>
               <td>Y</td>
     <td></td>
     <td>SmartDeviceLink 1.0</td>
   </tr>
   <tr>
   <td>position</td>
     <td>Integer</td>
     <td>Position within the items of the top level Command Menu. 0 will insert at the front, 1 will insert after the first existing element, etc. Position of any submenu will always be located before the return and exit options.</td>
               <td>N</td>
     <td>Min Value: 0 <br>Max Value: 1000<br> If position is greater or equal than the number of items on top level, the sub menu will be appended by the end.<br>If this parameter is omitted, the entry will be added at the end of the list.</td>
     <td>SmartDeviceLink 1.0</td>
   </tr>
   <tr>
     <td>menuName</td>
     <td>String</td>
     <td>Text which is displayed representing this submenu item</td>
               <td>Y</td>
     <td>maxlength:500</td>
     <td>SmartDeviceLink 1.0</td>
   </tr>
</table>

### Response ###

Indicates that the corresponding request either failed or succeeded. If the response returns with a SUCCESS result code, this means the SubMenu was added to the Command Menu successfully

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

Related Operations
- DeleteSubMenu
- AddCommand
- DeleteCommand

### Example Function Call ###
```java
AddSubMenu addSubMenu = new AddSubMenu(menuID, menuName);
proxy.sendRPCRequest(addSubMenu);
```