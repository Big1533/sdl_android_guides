## Cloud Apps


OEMs can create their own app stores to handle installing and uninstalling cloud apps. App stores can also handle user authentication for the installed cloud apps. For example, users can log in after installing a cloud app using the app store. After that, app store saves the authentication token for the cloud app in the local policy table. Then, the cloud app can retrieve the authentication token from the local policy table and use it to authenticate the websocket connection.


### Setting and Getting Cloud App Properties 
OEM's app store can manage the properties of a specific cloud app by setting and getting its `CloudAppProperties`. This table summarizes the properties that are included in `CloudAppProperties`.

| Parameter Name  |  Description |
| ------------- | ------------- |
| appID | appID for the cloud app |
| nicknames | List of possible names for the cloud app |
| enabled | If true, cloud app will be displayed on HMI |
| authToken | Used to authenticate websocket connection on app activation |
| cloudTransportType | Specifies the connection type Core should use |
| hybridAppPreference | Specifies the user preference to use the cloud app version or mobile app version when both are available |
| endpoint | Remote endpoint for websocket connections |

!!! note
Only trusted app stores are allowed to set or get `CloudAppProperties` for other cloud apps.
!!!


App stores can set cloud properties for a cloud app by sending `SetCloudAppProperties` request to Core to store the properties in the local policy table. For example, in this piece of code, the app store can set the `authToken` for a cloud app after the user logs in:

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

and to retrieve cloud properties for a specific cloud app from local policy table, app stores can send `GetCloudAppProperties` by specifying the `appId` for that cloud app as in this example:

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



### Retrieving Authentication Token

A cloud app can retrieve its `authToken` from local policy table after starting the RPC service. The `authToken` can be used later by the app to authenticate websocket connection on app activation:

```java
String authToken = sdlManager.getAuthToken();
```