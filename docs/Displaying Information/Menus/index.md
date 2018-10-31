## Menus
You have two different options when creating menus. One is to simply add items to the default menu available in every template. The other is to create a custom menu that pops up when needed.

### Default Menu
##### Ford HMI
![Menu Appearance](assets/MenuAppearance.png)  
Every template has a default menu button. The position of this button varies between templates, and can not be removed from the template. The default menu is initially empty except for an "Exit Your App Name" button. Items can be added to the menu at the root level or to a submenu. It is important to note that a submenu can only be one level deep.

### Menu Structure
![Menu Structure](assets/MenuStructure.png)

#### Add Menu Items
The `AddCommand` RPC can be used to add items to the root menu or to a submenu. Each `AddCommand` RPC must be sent with a unique id, a voice-recognition command, and a set of menu parameters. The menu parameters include the menu name, the position of the item in the menu, and the id of the menu item’s parent. If the menu item is being added to the root menu, then the parent id is 0. If it is being added to a submenu, then the parent id is the submenu’s id.

```java
// Create the menu parameters
// The parent id is 0 if adding to the root menu
// If adding to a submenu, the parent id is the submenu's id
MenuParams menuParams = new MenuParams();
menuParams.setParentID(0);
menuParams.setPosition(0);
menuParams.setMenuName("Options");

AddCommand addCommand = new AddCommand();
addCommand.setCmdID(0); // Ensure this is unique
addCommand.setMenuParams(menuParams);  // Set the menu parameters

sdlManager.sendRPC(addCommand);
```

#### Add a Submenu
To create a submenu, first send an `AddSubMenu` RPC. When a response is received from the SDL Core, check if the submenu was added successfully. If it was, send an `AddCommand` RPC for each item in the submenu.

```java
int unique_id = 313;

AddSubMenu addSubMenu = new AddSubMenu();
addSubMenu.setPosition(0);
addSubMenu.setMenuID(unique_id);
addSubMenu.setMenuName("SubMenu");
addSubMenu.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if(((AddSubMenuResponse) response).getSuccess()){
            // The submenu was created successfully, start adding the submenu items
            // Use unique_id
        }else{
            Log.i("SdlService", "AddSubMenu request rejected.");
        }
    }
});
```

#### Delete Menu Items
Use the cmdID of the menu item to tell the SDL Core which item to delete using the `DeleteCommand` RPC.

```java
int cmdID_to_delete = 1;

DeleteCommand deleteCommand = new DeleteCommand();
deleteCommand.setCmdID(cmdID_to_delete);
    
sdlManager.sendRPC(deleteCommand);
```

#### Delete Submenus
Use the menuID to tell the SDLCore which item to delete using the `DeleteSubMenu` RPC.

```java
DeleteSubMenu deleteSubMenu = new DeleteSubMenu();
deleteSubMenu.setMenuID(submenuID_to_delete); // Replace with submenu ID to delete
```  

### Custom Menus
![Perform Interaction Layout](assets/PerformInteractionListOnly.png)
Custom menus, called **perform interactions**, are one level deep, however, you can create submenus by triggering another perform interaction when the user selects a row in a menu. Perform interactions can be set up to recognize speech, so a user can select an item in the menu by speaking their preference rather than physically selecting the item.

 Perform interactions are created by sending two different RPCs. First a `CreateInteractionChoiceSet` RPC must be sent. This RPC sends a list of items that will show up in the menu. When the request has been registered successfully, then a `PerformInteraction` RPC is sent. The `PerformInteraction` RPC sends the formatting requirements, the voice-recognition commands, and a timeout command.

#### Create a Set of Custom Menu Items
Each menu item choice defined in `Choice` should be assigned a unique id. The choice set in `CreateInteractionChoiceSet` should also have its own unique id.

```java
CreateInteractionChoiceSet choiceSet = new CreateInteractionChoiceSet();

Choice choice = new Choice();
choice.setChoiceID(uniqueChoiceID);
choice.setMenuName("ChoiceA");
choice.setVrCommands(Arrays.asList("ChoiceA"));
	
List<Choice> choiceList = new ArrayList<>();
choiceList.add(choice);
	
choiceSet.setChoiceSet(choiceList);
choiceSet.setInteractionChoiceSetID(uniqueIntChoiceSetID);
choiceSet.setOnRPCResponseListener(new OnRPCResponseListener() {
@Override
public void onResponse(int correlationId, RPCResponse response) {
    if(((CreateInteractionChoiceSetResponse) response).getSuccess()){
        // The request was successful, now send the SDLPerformInteraction RPC
    }else{
        // The request was unsuccessful
    }
}
});
	
sdlManager.sendRPC(choiceSet);
```

#### Format the Set of Custom Menu Items
Once the set of menu items has been sent to SDL Core, send a `PerformInteraction` RPC to get the items to show up on the HMI screen.

```java
List<Integer> interactionChoiceSetIDList = new ArrayList<>();
interactionChoiceSetIDList.add(uniqueIntChoiceSetID);

PerformInteraction performInteraction = new PerformInteraction();
performInteraction.setInitialText("Initial text.");
performInteraction.setInteractionChoiceSetIDList(interactionChoiceSetIDList);
```

#### Interaction Mode
The interaction mode specifies the way the user is prompted to make a section and the way in which the user’s selection is recorded.

| Interaction Mode  | Description |
| ----------------- | ----------- |
| Manual only       | Interactions occur only through the display |
| VR only           | Interactions occur only through text-to-speech and voice recognition |
| Both              | Interactions can occur both manually or through VR |

```java
performInteraction.setInteractionMode(InteractionMode.MANUAL_ONLY);
```

#### VR Interaction Mode
##### Ford HMI
![VR Perform Interaction](assets/PerformInteractionVROnly.png)

#### Manual Interaction Mode
##### Ford HMI
![Manual Perform Interaction](assets/PerformInteractionManualOnly.png)

#### Interaction Layout
The items in the perform interaction can be shown as a grid of buttons (with optional images) or as a list of choices.

| Layout Mode      | Formatting Description |
| ---------------- | ---------------------- |
| Icon only        | A grid of buttons with images |
| Icon with search | A grid of buttons with images along with a search field in the HMI |
| List only        | A vertical list  of text |
| List with search | A vertical list of text with a search field in the HMI |
| Keyboard         | A keyboard shows up immediately in the HMI |

!!! note
Keyboard is currently only supported for the navigation app type.
!!!

```java
performInteraction.setInteractionLayout(LayoutMode.LIST_ONLY);
```

#### Icon Only Interaction Layout
##### Ford HMI
![Icon Only Interaction Layout](assets/PerformInteractionIconOnly.png)

#### List Only Interaction Layout
##### Ford HMI
![List Only Interaction Layout](assets/PerformInteractionListOnly.png)

#### List with Search Interaction Layout
##### Ford HMI
![List with Search Interaction Layout](assets/PerformInteractionListwithSearch.png)

#### Text-to-Speech (TTS)
A text-to-speech chunk is a text phrase or prerecorded sound that will be spoken by the head unit. The text parameter specifies the text to be spoken or the name of the pre-recorded sound. Use the type parameter to define the type of information in the text parameter. The `PerformInteraction` request can have a initial, timeout, and a help prompt.

```java
performInteraction.setInitialPrompt(
	TTSChunkFactory.createSimpleTTSChunks("Hello, welcome."));
```

#### Timeout
The timeout parameter defines the amount of time the menu will appear on the screen before the menu is dismissed automatically by the HMI.

```java
performInteraction.setTimeout(30000); // 30 seconds
```

#### Send the Request

```java
performInteraction.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        PerformInteractionResponse piResponse = (PerformInteractionResponse) response;
        if(piResponse.getSuccess()){
            // Successful request
            if(piResponse.getResultCode().equals(Result.TIMED_OUT)){
                // Interaction timed out without user input
            }else if(piResponse.getResultCode().equals(Result.SUCCESS)){
                Integer userChoice = piResponse.getChoiceID();
            }
        }else{
            // Unsuccessful request
        }
    }
});

sdlManager.sendRPC(performInteraction);
```

#### Delete the Custom Menu
If the information in the menu is dynamic, then the old interaction choice set needs to be deleted with a `DeleteInteractionChoiceSet` RPC before the new information can be added to the menu. Use the interaction choice set id to delete the menu.

```java
DeleteInteractionChoiceSet deleteInteractionChoiceSet = new DeleteInteractionChoiceSet();
deleteInteractionChoiceSet.setInteractionChoiceSetID(interactionChoiceSetID_to_delete); // Replace with interaction choice set to delete
    
sdlManager.sendRPC(deleteInteractionChoiceSet);
```
