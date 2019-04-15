## Creating a Cloud App Store
A new feature of SDL 5.1 allows OEMs to offer an app store that lets users browse and install remote cloud apps. If the cloud app requires users to login with their credentials, the app store can use an authentication token to automatically login users after their first session.

### User Authentication
App stores can handle user authentication for the installed cloud apps. For example, users can log in after installing a cloud app using the app store. After that, the app store will save an authentication token for the cloud app in the local policy table. Then, the cloud app can retrieve the authentication token from the local policy table and use it to match the websocket connection with a particular user. Optionally, `CloudAppVehicleID` can be used to identify the head unit.

!!! note
OEM app stores can be either mobile apps or cloud apps.
!!!

### Setting and Getting Cloud App Properties 
An OEM's app store can manage the properties of a specific cloud app by setting and getting its `CloudAppProperties`. This table summarizes the properties that are included in `CloudAppProperties`:

| Parameter Name  |  Description |
| ------------- | ------------- |
| appID | appID for the cloud app |
| nicknames | List of possible names for the cloud app. The cloud app will not be allowed to connect if its name is not contained in this list |
| enabled | If true, cloud app will be displayed on HMI |
| authToken | Used to authenticate websocket connection on app activation |
| cloudTransportType | Specifies the connection type Core should use. Currently Core supports WS and WSS , but an OEM can implement their own transport adapter to handle different values |
| hybridAppPreference | Specifies the user preference to use the cloud app version, mobile app version, or whichever connects first when both are available |
| endpoint | Remote endpoint for websocket connections |

!!! note
Only trusted app stores are allowed to set or get `CloudAppProperties` for other cloud apps.
!!!

#### Using SetCloudAppProperties
App stores can set cloud properties for a cloud app by sending `SetCloudAppProperties` request to Core to store the properties in the local policy table. For example, in this piece of code, the app store can set the `authToken` to associate a user with a login for a cloud app after the user logs in to the app using the app store:

```java
CloudAppProperties cloudAppProperties = new CloudAppProperties("<appId>");
cloudAppProperties.setAuthToken("<auth token>");
SetCloudAppProperties setCloudAppProperties = new SetCloudAppProperties(cloudAppProperties);
setCloudAppProperties.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (response.getSuccess()) {
            Log.i("SdlService", "Request was successful.");
        } else {
            Log.i("SdlService", "Request was rejected.");
        }
    }
});
sdlManager.sendRPC(setCloudAppProperties);
```

#### Using SetCloudAppProperties
To retrieve cloud properties for a specific cloud app from local policy table, app stores can send `GetCloudAppProperties` and specify the `appId` for that cloud app as in this example:

```java
GetCloudAppProperties getCloudAppProperties = new GetCloudAppProperties("<appId>");
getCloudAppProperties.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (response.getSuccess()) {
            Log.i("SdlService", "Request was successful.");
            GetCloudAppPropertiesResponse getCloudAppPropertiesResponse = (GetCloudAppPropertiesResponse) response;
            CloudAppProperties cloudAppProperties = getCloudAppPropertiesResponse.getCloudAppProperties();
            // Use cloudAppProperties
        } else {
            Log.i("SdlService", "Request was rejected.");
        }
    }
});
sdlManager.sendRPC(getCloudAppProperties);
```

### Getting the Cloud App Icon
The cloud app icon will automatically be downloaded from the url provided by the policy table and sent to Core to be later displayed on HMI.

### Retrieving the Authentication Token
When users install cloud apps from an OEM's app store, they may be asked to login to that cloud app using the app store. After login in, app store can save the `authToken` in the local policy table to be used later by cloud app for user authentication. 
A cloud app can retrieve its `authToken` from local policy table after starting the RPC service. The `authToken` can be used later by the app to authenticate websocket connection on app activation:

```java
String authToken = sdlManager.getAuthToken();
```

### Using CloudAppVehicleID (Optional)
The `CloudAppVehicleID` is an optional parameter used by cloud apps to identify a head unit. The content of `CloudAppVehicleID` is up to the OEM's implementation. Possible values could be the VIN or a hashed VIN. 
`CloudAppVehicleID` value can be retrieved as part of the `GetVehicleData` RPC.  To find out more about how to retrieve `CloudAppVehicleID`, check out the  [Getting Vehicle Data](Getting Vehicle Data) section.