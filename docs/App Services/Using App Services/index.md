# Using App Services
App services is a powerful feature enabling both a new kind of vehicle-to-app communication and app-to-app communication via SDL.

App services are used to publish navigation, weather and media data (such as temperature, navigation waypoints, or the current playlist name). This data can then be used by both the vehicle head unit and, if the publisher of the app service desires, other SDL apps. Creating an app service is covered [in another guide](App Services/Creating an App Service).

Vehicle head units may use these services in various ways. One app service for each type will be the "active" service to the module. For media, for example, this will be the media app that the user is currently using or listening to. For navigation, it would be a navigation app that the user is using to navigate. For weather, it may be the last used weather app, or a user-selected default. The system may then use that service's data to perform various actions (such as navigating to an address with the active service or to display the temperature as provided from the active weather service).

An SDL app can also subscribe to a published app service. Once subscribed, the app will be sent the new data when the app service publisher updates its data. This guide will cover subscribing to a service. Subscribed apps can also send certain RPCs and generic URI-based actions (see the section Supporting App Actions, below) to your service.

Currently, there is no high-level API support for using an app service, so you will have to use raw RPCs for all app service related APIs.

## Getting and Subscribing to Services
Once your app has connected to the head unit, you will first want to be notified of all available services and updates to the metadata of all services on the head unit. Second, you will narrow down your app to subscribe to an individual app service and subscribe to its data. Third, you may want to interact with that service through RPCs, or fourth, through service actions.

### 1. Getting and Subscribing to Available Services
To get information on all services published on the system, as well as on changes to published services, you will use the `SystemCapabilityManager` to get the information. Because this information is initially available asynchronosly, we have to attach an `OnSystemCapabilityListener` to the `getCapability` request.

##### Java
```java
// Grab the capability once
sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.APP_SERVICES, new OnSystemCapabilityListener() {
    @Override
    public void onCapabilityRetrieved(Object capability) {
        AppServicesCapabilities servicesCapabilities = (AppServicesCapabilities) capability;
    }

    @Override
    public void onError(String info) {
        <# Handle Error #>
    }
});

...

// Subscribe to updates
sdlManager.getSystemCapabilityManager().addOnSystemCapabilityListener(SystemCapabilityType.APP_SERVICES, new OnSystemCapabilityListener() {
    @Override
    public void onCapabilityRetrieved(Object capability) {
        AppServicesCapabilities servicesCapabilities = (AppServicesCapabilities) capability;
    }

    @Override
    public void onError(String info) {
        <# Handle Error #>
    }
});
```

#### Checking the App Service Capability
Once you've retrieved the initial list of app service capabilities (in the `GetSystemCapability` response), or an updated list of app service capabilities (from the `OnSystemCapabilityUpdated` notification), you may want to inspect the data to find what you are looking for. Below is example code with comments explaining what each part of the app service capability is used for.

##### Java
```java
// This array contains all currently available app services on the system
List<AppServiceCapability> appServices = servicesCapabilities.getAppServices();

if (appServices!= null && appServices.size() > 0) {
    for (AppServiceCapability anAppServiceCapability : appServices) {
        // This will tell you why a service is in the list of updates
        ServiceUpdateReason updateReason = anAppServiceCapability.getUpdateReason();

        // The app service record will give you access to a service's generated id, which can be used to address the service directly (see below), it's manifest, used to see what data it supports, whether or not the service is published (it always will be here), and whether or not the service is the active service for its service type (only one service can be active for each type)
        AppServiceRecord serviceRecord = anAppServiceCapability.getUpdatedAppServiceRecord();
    }
}
```

### 2. Getting and Subscribing to a Service Type's Data
Once you have information about all of the services available, you may want to view or subscribe to a service type's data. To do so, you will use the `GetAppServiceData` RPC.

Note that you will currently only be able to get data for the *active* service of the service type. You can attempt to make another service the active service by using the `PerformAppServiceInteraction` RPC, discussed below in "Sending an Action to a Service Provider."

##### Java
```java
// Get service data once
GetAppServiceData getAppServiceData = new GetAppServiceData(AppServiceType.MEDIA.toString());

// Subscribe to future updates if you want them
getAppServiceData.setSubscribe(true);

getAppServiceData.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (response != null){
            GetAppServiceDataResponse serviceResponse = (GetAppServiceDataResponse) response;
            MediaServiceData mediaServiceData = serviceResponse.getServiceData().getMediaServiceData();
        }
    }
    @Override
    public void onError(int correlationId, Result resultCode, String info){
        <# Handle Error #>
    }
});
sdlManager.sendRPC(getAppServiceData);

...

// Unsubscribe from updates
GetAppServiceData unsubscribeServiceData = new GetAppServiceData(AppServiceType.MEDIA.toString());
unsubscribeServiceData.setSubscribe(false);
sdlManager.sendRPC(unsubscribeServiceData);
```

## Interacting with a Service Provider
Once you have a service's data, you may want to interact with a service provider by sending RPCs or actions.

### 3. Sending RPCs to a Service Provider
Only certain RPCs are available to be passed to the service provider based on their service type. See the [Creating an App Service](App Services/Creating an App Service) guide (under the "Supporting Service RPCs and Actions" section) for a chart detailing which RPCs work with which service types. The RPC can only be sent to the active service of a specific service type, not to any inactive service.

Sending an RPC works exactly the same as if you were sending the RPC to the head unit system. The head unit will simply route your RPC to the appropriate app automatically.

!!! NOTE
Your app may need special permissions to use the RPCs that route to app service providers.
!!!

##### Java
```java
ButtonPress buttonPress = new ButtonPress();
buttonPress.setButtonPressMode(ButtonPressMode.SHORT);
buttonPress.setButtonName(ButtonName.OK);
buttonPress.setModuleType(ModuleType.AUDIO);
buttonPress.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        <#Use the response#>
    }
    @Override
    public void onError(int correlationId, Result resultCode, String info){
        <#Handle the Error#>
    }
});
sdlManager.sendRPC(buttonPress);
```

### 4. Sending an Action to a Service Provider
Actions are generic URI-based strings sent to any app service (active or not). You can also use actions to request to the system that they make the service the active service for that service type. Service actions are *schema-less*, i.e. there is no way to define the appropriate URIs through SDL. The service provider must document their list of available actions elsewhere (such as their website).

##### Java
```java
PerformAppServiceInteraction performAppServiceInteraction = new PerformAppServiceInteraction("sdlexample://x-callback-url/showText?x-source=MyApp&text=My%20Custom%20String","<#Previously Retrieved ServiceID#>","<#Your App Id#>");
performAppServiceInteraction.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        <#Use the response#>
    }
    @Override
    public void onError(int correlationId, Result resultCode, String info){
        <#Handle the Error#>
    }
});
sdlManager.sendRPC(performAppServiceInteraction);
```
