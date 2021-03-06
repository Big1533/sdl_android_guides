# Creating an App Service
App services is a powerful feature enabling both a new kind of vehicle-to-app communication and app-to-app communication via SDL.

App services are used to publish navigation, weather and media data (such as temperature, navigation waypoints, or the current playlist name). This data can then be used by both the vehicle head unit and, if the publisher of the app service desires, other SDL apps.  

Vehicle head units may use these services in various ways. One app service for each type will be the "active" service to the module. For media, for example, this will be the media app that the user is currently using or listening to. For navigation, it would be a navigation app that the user is using to navigate. For weather, it may be the last used weather app, or a user-selected default. The system may then use that service's data to perform various actions (such as navigating to an address with the active service or to display the temperature as provided from the active weather service).

An SDL app can also subscribe to a published app service. Once subscribed, the app will be sent the new data when the app service publisher updates its data. To find out more about how to subscribe to an app service check out the [Using App Services](App Services/Using App Services) section. Subscribed apps can also send certain RPCs and generic URI-based actions (see the section Supporting App Actions, below) to your service.

Currently, there is no high-level API support for publishing an app service, so you will have to use raw RPCs for all app service related APIs.

Using an app service is covered [in another guide](App Services/Using App Services).

## App Service Types
Apps are able to declare that they provide an app service by publishing an app service manifest. Three types of app services are currently available, and more will be made available over time. The currently available types are: Media, Navigation, and Weather. An app may publish multiple services (one each for different service types), if desired.

## Publishing an App Service
Publishing a service is a several step process. First, create your app service manifest. Second, publish your app service using your manifest. Third, publish your service data using `OnAppServiceData`. Fourth, respond to `GetAppServiceData` requests. Fifth, you should support RPCs related to your service. Last, optionally, you can support URI based app actions.

### 1. Creating an App Service Manifest
The first step to publishing an app service is to create an `AppServiceManifest` object. There is a set of generic parameters you will need to fill out as well as service type specific parameters based on the app service type you are creating.

##### Java
```java
AppServiceManifest manifest = new AppServiceManifest(AppServiceType.MEDIA.toString());
manifest.setServiceName("My Media App"); // Must be unique across app services.
manifest.setServiceIcon(new Image("Service Icon Name", ImageType.DYNAMIC)); // Previously uploaded service icon. This could be the same as your app icon.
manifest.setAllowAppConsumers(true); // Whether or not other apps can view your data in addition to the head unit. If set to `NO` only the head unit will have access to this data.
manifest.setRpcSpecVersion(new SdlMsgVersion(5,0)); // An *optional* parameter that limits the RPC spec versions you can understand to the provided version *or below*.
manifest.setHandledRpcs(List<FunctionID>); // If you add function ids to this *optional* parameter, you can support newer RPCs on older head units (that don't support those RPCs natively) when those RPCs are sent from other connected applications.
manifest.setMediaServiceManifest(<#Code#>); // Covered Below
```

#### Creating a Media Service Manifest
Currently, there's no information you have to provide in your media service manifest! You'll just have to create an empty media service manifest and set it into your general app service manifest.

##### Java
```java
MediaServiceManifest mediaManifest = new MediaServiceManifest();
manifest.setMediaServiceManifest(mediaManifest);
```

#### Creating a Navigation Service Manifest
You will need to create a navigation manifest if you want to publish a navigation service. You will declare whether or not your navigation app will accept waypoints. That is, if your app will support receiving _multiple_ points of navigation (e.g. go to this McDonalds, then this Walmart, then home).

##### Java
```java
NavigationServiceManifest navigationManifest = new NavigationServiceManifest();
navigationManifest.setAcceptsWayPoints(true);
manifest.setNavigationServiceManifest(navigationManifest);
```

#### Creating a Weather Service Manifest
You will need to create a weather service manifest if you want to publish a weather service. You will declare the types of data your service provides in its `WeatherServiceData`.

##### Java
```java
WeatherServiceManifest weatherManifest = new WeatherServiceManifest();
weatherManifest.setCurrentForecastSupported(true);
weatherManifest.setMaxMultidayForecastAmount(10);
weatherManifest.setMaxHourlyForecastAmount(24);
weatherManifest.setMaxMinutelyForecastAmount(60);
weatherManifest.setWeatherForLocationSupported(true);
manifest.setWeatherServiceManifest(weatherManifest);
```

### 2. Publish Your Service
Once you have created your service manifest, publishing your app service is simple.

##### Java
```java
PublishAppService publishServiceRequest = new PublishAppService();
publishServiceRequest.setAppServiceManifest(manifest);
publishServiceRequest.setOnRPCResponseListener(new OnRPCResponseListener() {
	@Override
	public void onResponse(int correlationId, RPCResponse response) {
		<#Use the response#>
	}

	@Override
	public void onError(int correlationId, Result resultCode, String info){
		<#Error Handling#>
	}
});
sdlManager.sendRPC(publishServiceRequest);
```

Once you have your publish app service response, you will need to store the information provided in its `appServiceRecord` property. You will need the information later when you want to update your service's data.

#### Watching for App Record Updates
As noted in the introduction to this guide, one service for each type may become the "active" service. If your service is the active service, your `AppServiceRecord` parameter `serviceActive` will be updated to note that you are now the active service.

After the initial app record is passed to you in the `PublishAppServiceResponse`, you will need to be notified of changes in order to observe whether or not you have become the active service. To do so, you will have to observe the new `SystemCapabilityType.APP_SERVICES` using `GetSystemCapability` and `OnSystemCapabilityUpdated`.

For more information, see the [Using App Services guide](App Services/Using App Services) and see the "Getting and Subscribing to Services" section.

### 3. Update Your Service's Data
After your service is published, it's time to update your service data. First, you must send an `onAppServiceData` RPC notification with your updated service data. RPC notifications are different than RPC requests in that they will not receive a response from the connected head unit.

!!! NOTE
You should only update your service's data when you are the active service; service consumers will only be able to see your data when you are the active service.
!!!

First, you will have to create an `MediaServiceData`, `NavigationServiceData` or `WeatherServiceData` object with your service's data. Then, add that service-specific data object to an `AppServiceData` object. Finally, create an `OnAppServiceData` notification, append your `AppServiceData` object, and send it.

#### Media Service Data

##### Java
```java
MediaServiceData mediaData = new MediaServiceData();
mediaData.setMediaTitle("Some media title");
mediaData.setMediaArtist("Some media artist");
mediaData.setMediaAlbum("Some album");
mediaData.setPlaylistName("Some playlist");
mediaData.setIsExplicit(true);
mediaData.setTrackPlaybackProgress(45);
mediaData.setQueuePlaybackDuration(90);
mediaData.setTrackPlaybackProgress(45);
mediaData.setQueuePlaybackDuration(150);
mediaData.setQueueCurrentTrackNumber(2);
mediaData.setQueueTotalTrackCount(3);

AppServiceData appData = new AppServiceData();
appData.setServiceID(myServiceId);
appData.setServiceType(AppServiceType.MEDIA.toString());
appData.setMediaServiceData(mediaData);

OnAppServiceData onAppData = new OnAppServiceData();
onAppData.setServiceData(appData);
		
sdlManager.sendRPC(onAppData);
```

#### Navigation Service Data

##### Java
```java
final SdlArtwork navInstructionArt = new SdlArtwork("turn", FileType.GRAPHIC_PNG, R.drawable.turn, true);
        
sdlManager.getFileManager().uploadFile(navInstructionArt, new CompletionListener() { // We have to send the image to the system before it's used in the app service.
    @Override
    public void onComplete(boolean success) {
        if (success){
            Coordinate coordinate = new Coordinate(42f,43f);

            LocationDetails locationDetails = new LocationDetails();
            locationDetails.setCoordinate(coordinate);

            NavigationInstruction navigationInstruction = new NavigationInstruction(locationDetails, NavigationAction.TURN);
            navigationInstruction.setImage(navInstructionArt.getImageRPC());

            DateTime dateTime = new DateTime();
            dateTime.setHour(2);
            dateTime.setMinute(3);
            dateTime.setSecond(4);

            NavigationServiceData navigationData = new NavigationServiceData(dateTime);
            navigationData.setInstructions(Collections.singletonList(navigationInstruction));

            AppServiceData appData = new AppServiceData();
            appData.setServiceID(myServiceId);
            appData.setServiceType(AppServiceType.NAVIGATION.toString());
            appData.setNavigationServiceData(navigationData);

            OnAppServiceData onAppData = new OnAppServiceData();
            onAppData.setServiceData(appData);

            sdlManager.sendRPC(onAppData);
        }
    }
});
```

#### Weather Service Data

##### Java
```java
final SdlArtwork weatherImage = new SdlArtwork("sun", FileType.GRAPHIC_PNG, R.drawable.sun, true);

sdlManager.getFileManager().uploadFile(weatherImage, new CompletionListener() { // We have to send the image to the system before it's used in the app service.
    @Override
    public void onComplete(boolean success) {
        if (success) {

            WeatherData weatherData = new WeatherData();
            weatherData.setWeatherIcon(weatherImage.getImageRPC());

            Coordinate coordinate = new Coordinate(42f, 43f);

            LocationDetails locationDetails = new LocationDetails();
            locationDetails.setCoordinate(coordinate);

            WeatherServiceData weatherServiceData = new WeatherServiceData(locationDetails);

            AppServiceData appData = new AppServiceData();
            appData.setServiceID(myServiceId);
            appData.setServiceType(AppServiceType.WEATHER.toString());
            appData.setWeatherServiceData(weatherServiceData);

            OnAppServiceData onAppData = new OnAppServiceData();
            onAppData.setServiceData(appData);

            sdlManager.sendRPC(onAppData);
        }
    }
});
```

### 4. Handling App Service Subscribers
If you choose to make your app service available to other apps, you will have to handle requests to get your app service data when a consumer requests it directly.

Handling app service subscribers is a two step process. First, you must register for notifications from the subscriber. Then, when you get a request, you will either have to send a response to the subscriber with the app service data or if you have no data to send, send a reponse with a relevant failure result code.

#### Listening for Requests
First, you will need to register for updates for when a `GetAppServiceDataRequest` is received by your application.

##### Java
```java
// Get App Service Data Request Listener
sdlManager.addOnRPCRequestListener(FunctionID.GET_APP_SERVICE_DATA, new OnRPCRequestListener() {
    @Override
    public void onRequest(RPCRequest request) {
        <#Handle Request#>
    }
});
```

#### Sending a Response to Subscribers
Second, you need to respond to the request when you receive it with your app service data. This means that you will need to store your current service data after your most recent update using `OnAppServiceData` (see the section Updating Your Service Data).

##### Java
```java
// Get App Service Data Request Listener
sdlManager.addOnRPCRequestListener(FunctionID.GET_APP_SERVICE_DATA, new OnRPCRequestListener() {
    @Override
    public void onRequest(RPCRequest request) {
        GetAppServiceData getAppServiceData = (GetAppServiceData) request;

        GetAppServiceDataResponse response = new GetAppServiceDataResponse();
        response.setSuccess(true);
        response.setCorrelationID(getAppServiceData.getCorrelationID());
        response.setResultCode(Result.SUCCESS);
        response.setInfo("<#Use to provide more information about an error#>");
        response.setServiceData(<#Your App Service Data#>);

        sdlManager.sendRPC(response);
    }
});
```

## Supporting Service RPCs and Actions

### 5. Service RPCs
Certain RPCs are related to certain services. The chart below shows the current relationships:

| MEDIA | NAVIGATION | WEATHER |
| ----- | ---------- | ------- |
| ButtonPress (OK) | SendLocation | |
| ButtonPress (SEEKLEFT) | GetWayPoints | |
| ButtonPress (SEEKRIGHT) | SubscribeWayPoints | |
| ButtonPress (TUNEUP) | OnWayPointChange | |
| ButtonPress (TUNEDOWN) | | |
| ButtonPress (SHUFFLE) | | |
| ButtonPress (REPEAT) | | |

When you are the active service for your service's type (e.g. media), and you have declared that you support these RPCs in your manifest (see section 1. Creating an App Service Manifest), then these RPCs will be automatically routed to your app. You will have to set up listeners to be aware that they have arrived, and you will then need to respond to those requests.

##### Java
```java
AppServiceManifest manifest = new AppServiceManifest(AppServiceType.MEDIA.toString());
...
manifest.setHandledRpcs(Collections.singletonList(FunctionID.BUTTON_PRESS.getId()));
```

##### Java
```java
sdlManager.addOnRPCRequestListener(FunctionID.BUTTON_PRESS, new OnRPCRequestListener() {
    @Override
    public void onRequest(RPCRequest request) {
        ButtonPress buttonPress = (ButtonPress) request;

        ButtonPressResponse response = new ButtonPressResponse();
        response.setSuccess(true);
        response.setResultCode(Result.SUCCESS);
        response.setCorrelationID(buttonPress.getCorrelationID());
        response.setInfo("<#Use to provide more information about an error#>");
        sdlManager.sendRPC(response);
    }
});
```

### 6. Service Actions
App actions are the ability for app consumers to use the SDL services system to send URIs to app providers in order to activate actions on the provider. Service actions are *schema-less*, i.e. there is no way to define the appropriate URIs through SDL. If you already provide actions through your app and want to expose them to SDL, or if you wish to start providing them, you will have to document your available actions elsewhere (such as your website).

In order to support actions through SDL services, you will need to observe and respond to the `PerformAppServiceInteraction` RPC request.

##### Java
```java
// Perform App Services Interaction Request Listener
sdlManager.addOnRPCRequestListener(FunctionID.PERFORM_APP_SERVICES_INTERACTION, new OnRPCRequestListener() {
    @Override
    public void onRequest(RPCRequest request) {
        PerformAppServiceInteraction performAppServiceInteraction = (PerformAppServiceInteraction) request;

        // If you have multiple services, this will let you know which of your services is being addressed
        serviceID = performAppServiceInteraction.getServiceID();

        // The URI sent by the consumer. This must be something you understand
        String serviceURI = performAppServiceInteraction.getServiceUri();

        // A result you want to send to the consumer app.
        PerformAppServiceInteractionResponse response = new PerformAppServiceInteractionResponse();
        response.setServiceSpecificResult("Some Result");
        response.setCorrelationID(performAppServiceInteraction.getCorrelationID());
        response.setInfo("<#Use to provide more information about an error#>");
        response.setSuccess(true);
        response.setResultCode(Result.SUCCESS);
        sdlManager.sendRPC(response);
    }
});
```
