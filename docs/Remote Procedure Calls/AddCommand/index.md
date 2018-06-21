# AddCommand

Adds a Command to the application's Command Menu.
A Command will be added to the end of the list of elements in the Command Menu under the following conditions:

- When a Command is added with no MenuParams value provided
- When a MenuParams value is provided with a MenuParam.position value greater than or equal to the number of menu items currently defined in the menu specified by the MenuParam.parentID value

The set of choices which the application builds using AddCommand can be a mixture of:

- Choices having only VR synonym definitions, but no MenuParams definitions
- Choices having only MenuParams definitions, but no VR synonym definitions
- Choices having both MenuParams and VR synonym definitions

When a user initiates an interaction, the user may choose from whatever choices are defined at that moment. It is up to the application to ensure that all appropriate choices are defined before the user initiates an interaction.

**Note:**  _A consequence of this is that it is possible for the application to be in the middle of adding commands (or submenu items) when the user presses the PTT or Menu button and only the commands added up to that moment will be available for the user to choose from (either by VR or Menu). It is possible in this situation for the user to not have had all intended options available when they started the interaction. The application will receive an OnHMIStatus with a SystemContext value of VRSESSION or MENU indicating that such a user-initiated interaction has begun. The application can make use of this notification to know that, if it is still building a Command Menu using AddCommand/AddSubMenu, the results of the interaction should be discarded. This approach is necessary because there is no way to "cancel" an interaction in progress (neither a user initiated interaction, nor an app-initiated interaction)._

**Note:** _smartdevicelink™ batches together AddCommand and AddSubMenu requests before making them available for selection by the user via PTT or the menu. The end of a batch of successive AddCommand/AddSubMenu requests is defined as 500 milliseconds passing with no further AddCommand/AddSubMenu requests arriving at SDL®. When the batch of requests has ended, SDL® then prepares these commands for use by the user. A command should be available for use 1 second after the end of the batch in which that command was added (timing based batch size)._

**Note:** Commands should be “grouped” by using position id. If you are planning to create child commands of a parent command to keep the commands grouped together. Add the commands to the top level menu (basically the root); you can pass in a 0 for the parentID. If you would like to add the commands to a submenu, then the submenuID is passed in for the parentID.

The position ID can be used to sort the commands in the menu. However, the logic isn't always straightforward. The position is relative to the commands that are currently in existence. However, once commands are added, they drop the Position parameter and the sync hardware just tracks their sequential order.

For example: send add: command1, Position 5 send add: command2, Position 10 send add: command3, Position 20

Results in on tdk: command1, command2, command3

If you then followed up with: send add: command4, Position 1

You get the following result on the TDK: command1, command4, command2, command3


There are a few other noteworthy consequences of MenuParams for a given command:
- Commands that do not have vrCommands associated with them will not be accessible by voice commands (when the user hits push-to-talk).
- Commands that do not have menuParams associated with them will not be accessible through the HMI application menu

### HMI Status Requirements
HMILevel needs to be FULL, LIMITED, or BACKGROUND.

## Request
**Parameter List**
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
   <td>cmdID</td>
   <td> Integer</td>
   <td>unique ID of the command to add</td>
              <td>Y</td>
   <td> minvalue:0<br>maxvalue:2000000000</td>
   <td>SmartDeviceLink 1.0</td>
 </tr>
 <tr>
   <td>menuParams</td>
   <td>ButtonName</td>
   <td>Name of the button to unsubscribe.</td>
              <td>Y</td>
   <td></td>
   <td>SmartDeviceLink 1.0</td>
 </tr>
 <tr>
   <td>vrCommands</td>
   <td>String</td>
   <td>An array of strings to be used as VR synonyms for this command.<br>    	If this array is provided, it may not be empty.</td>
              <td>N</td>
   <td>minsize:1<br> maxsize:100</td>
   <td>SmartDeviceLink 1.0</td>
 </tr>
 <tr>
   <td>cmdIcon</td>
   <td>Image</td>
   <td>Image struct determining whether static or dynamic icon.<br>If omitted on supported displays, no (or the default if applicable) icon shall be displayed.</td>
              <td>N</td>
   <td></td>
   <td>SmartDeviceLink 1.0</td>
 </tr>
</table>

### Response

Indicates that the corresponding request has failed or succeeded, if the response returns with a SUCCESS result code, this means a command was added to the Command Menu successfully.

Non-default Result Codes:

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
- DISALLOWED

### Related Operations 
- DeleteCommand
- AddSubMenu
- DeleteSubMenu

### Example Function Call 
```java
AddCommand addCommand = new AddCommand(100);
addCommand.setMenuParams(new MenuParams("Skip"));
proxy.sendRPCRequest(addCommand);
```