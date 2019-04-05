## Cloud Apps

### Cloud App Properties 
App stores need to manage the `CloudAppProperties` for cloud apps in the policy table in order to enable/disable a cloud app from appearing on the HMI or deliver the necessary authentication information if the app requires it. This table includes the properties that are included in `CloudAppProperties`.

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


App stores can set cloud properties for a cloud app by sending `SetCloudAppProperties` request to Core to store the properties in the policy table. For example, in this piece of code, the app store can set the `authToken` for a cloud app after the user logs in:

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

Also, app stores can retrieve cloud properties for a specific cloud app from policy table by sending `GetCloudAppProperties` and specifying the `appId` for that cloud app as in this example:

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



### Auth Token

A Cloud apps can retrieve its `authToken` from policy table after the app successfully starts the RPC service by using the `SdlManager` as in the following example:

```java
String authToken = sdlManager.getAuthToken();
```