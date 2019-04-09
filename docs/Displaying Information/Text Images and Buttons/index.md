## Text, Images, and Buttons

All text, images, and soft buttons on the HMI screen must be sent as part of a `Show` RPC. The `ScreenManager` will take care of creating and sending the `Show` request for text, images, and soft buttons so developers don't have to worry about that. Subscribe buttons need to be sent as part of a `SubscribeButton` RPC.

### Text

A maximum of four lines of text can be sent to the module, however, some templates may only support 1, 2, or 3 lines of text. The `ScreenManager` well automatically handle the combining of lines based on how many lines are available and which fields the developer has set. For example, if all four lines of text are set in the `ScreenManager`, but the template only supports three lines of text, then the `ScreenManager` will hyphenate the third and fourth line and display them in one line.

```java
//Start the UI updates
sdlManager.getScreenManager().beginTransaction();

sdlManager.getScreenManager().setTextField1("Hello, this is MainField1.");
sdlManager.getScreenManager().setTextField2("Hello, this is MainField2.");
sdlManager.getScreenManager().setTextField3("Hello, this is MainField3.");
sdlManager.getScreenManager().setTextField4("Hello, this is MainField4.");

//Commit the UI updates
sdlManager.getScreenManager().commit(new CompletionListener() {
	@Override
	public void onComplete(boolean success) {
		Log.i(TAG, "ScreenManager update complete: " + success);
	}
});
```

!!! NOTE
If you don't use `beginTransaction()` and `commit()`, `ScreenManager` will still update the text fields correctly, however, it will send a `Show` request every time a text field is set. It is always recommended to use transactions if you have a batch of `ScreenManager` updates. Transactions will let the `ScreenManager` queue the updates and send them all at once in one `Show` RPC when `commit()` is called resulting in better performance and UI stability.
!!!

### Images

The position and size of images on the screen is determined by the currently set template. `ScreenManager` will handle uploading images and sending the `Show` RPC to display the images when they are ready.

!!! NOTE
Some head units may only support certain images or possibly none at all. Please consult the `getGraphicSupported()` method in the `DisplayCapabilities` using the `SystemCapabilityManager`.
!!!

#### Show the Image on a Head Unit

To display an image in the head unit, you have to create an `SdlArtwork` object and set it using the `ScreenManager`. `SdlArtwork` supports both dynamic and static images. 
To use it with dynamic images, you can use the constructor that takes multiple arguments. The `fileName` property should be set to the name that you want to use to save the file in the head unit. The `FileType` should be set to the correct type of image that is being sent, in the example it is set to `FileType.GRAPHIC_JPEG` because the image has JPEG format. The `id` is set to the Android resource id of the image that you want to use. The `persistentFile` is a boolean that represents whether you want the file to persist between sessions.

```java
SdlArtwork sdlArtwork = new SdlArtwork("appImage.jpeg", FileType.GRAPHIC_JPEG, R.drawable.appImage, true);
sdlManager.getScreenManager().setPrimaryGraphic(sdlArtwork);
```

To use `SdlArtwork` with static images, you can use the constructor that takes the static icon name as the only argument as in the following sample:

```java
SdlArtwork sdlArtwork = new SdlArtwork(StaticIconName.ALBUM);
sdlManager.getScreenManager().setPrimaryGraphic(sdlArtwork);
```

### Soft & Subscribe Buttons

Buttons pushed by an app to the module's HMI screen are referred to as soft buttons to distinguish them from hard or preloaded buttons, which are either physical buttons on the head unit or buttons that exist on the module at all times. Don’t confuse soft buttons with subscribe buttons, which are buttons that can detect user selection on hard buttons (or built-in soft buttons).

#### Soft Buttons

Soft buttons can be created with text, images or both text and images. The location, size, and number of soft buttons visible on the screen depends on the template. A `SoftButtonObject` can have multiple `SoftButtonState` objects; each state can have text, image, or both. Buttons can be transitioned from one state to another at runtime.

```java
SoftButtonState softButtonState1 = new SoftButtonState("state1", "state1", new SdlArtwork("state1.png", FileType.GRAPHIC_PNG, R.drawable.state1, true));

SoftButtonState softButtonState2 = new SoftButtonState("state2", "state2", new SdlArtwork("state2.png", FileType.GRAPHIC_PNG, R.drawable.state2, true));

List<SoftButtonState> softButtonStates = Arrays.asList(softButtonState1, softButtonState2);
SoftButtonObject softButtonObject = new SoftButtonObject("object", softButtonStates, softButtonState1.getName(), null);

//We will add a listener for events in the next example here 

sdlManager.getScreenManager().setSoftButtonObjects(Collections.singletonList(softButtonObject));
```


##### Receiving Soft Buttons Events

Once you have created soft buttons, you will likely want to know when events happen to those buttons. These events come through two callbacks `onEvent` and `onPress `. Depending which type of event you're looking for you can use that type of callback. 

```java
softButtonObject.setOnEventListener(new SoftButtonObject.OnEventListener() {
    @Override
    public void onPress(SoftButtonObject softButtonObject, OnButtonPress onButtonPress) {
        softButtonObject.transitionToNextState();
    }

    @Override
    public void onEvent(SoftButtonObject softButtonObject, OnButtonEvent onButtonEvent) {

    }
});
```

#### Subscribe Buttons
Subscribe buttons are used to detect changes to hard or preloaded buttons. You can subscribe to the following hard buttons:

| Button  | Template | Button Type |
| ------------- | ------------- | ------------- |
| Ok (play/pause) | media template only | soft button and hard button |
| Seek left | media template only | soft button and hard button |
| Seek right | media template only | soft button and hard button |
| Tune up | media template only | hard button |
| Tune down | media template only | hard button |
| Preset 0-9 | any template | hard button |
| Search | any template |hard button |
| Custom | any template | hard button |

Audio buttons like the OK (i.e. the `play/pause` button), seek left, seek right, tune up, and tune down buttons can only be used with a media template. The OK, seek left, and seek right buttons will also show up on the screen in a predefined location dictated by the media template on touchscreens. The app will be notified when the user selects the subscribe button on the screen or when the user manipulates the corresponding hard button.

You can subscribe to buttons using the `SubscribeButton` RPC. 

```java
SubscribeButton subscribeButtonRequest = new SubscribeButton();
subscribeButtonRequest.setButtonName(ButtonName.SEEKRIGHT);
sdlManager.sendRPC(subscribeButtonRequest);
```

!!! NOTE
It is not required to manually subscribe to soft buttons. When soft buttons are added, your app will automatically be subscribed for their events.
!!!

##### Receiving Subscribe Buttons Events

When you want to subscribe to buttons, you will be subscribing to events that happen to those buttons. These events come through two callbacks `OnButtonEvent` and `OnButtonPress `. Depending which type of event you're looking for you can use that type of callback. The `ButtonName` enum refers to which button the event happened to.

!!! NOTE
Some templates will not show a preloaded button until an app subscribes to it. After an app subscribes to the events of that button, it will appear. 
!!!

```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_BUTTON_EVENT, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnButtonPress onButtonPressNotification = (OnButtonPress) notification;
        switch (onButtonPressNotification.getButtonName()) {
            case OK:
                break;
            case SEEKLEFT:
                break;
            case SEEKRIGHT:
                break;
            case TUNEUP:
                break;
            case TUNEDOWN:
                break;
            default:
                break;
        }
    }
});


sdlManager.addOnRPCNotificationListener(FunctionID.ON_BUTTON_PRESS, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnButtonPress onButtonPressNotification = (OnButtonPress) notification;
        switch (onButtonPressNotification.getButtonName()) {
            case OK:
                break;
            case SEEKLEFT:
                break;
            case SEEKRIGHT:
                break;
            case TUNEUP:
                break;
            case TUNEDOWN:
                break;
            default:
                break;
        }
    }
});
```

!!! NOTE
The app should subscribe to button events before sending the `SubscribeButton` request to make sure that it doesn't miss any button events.
!!!