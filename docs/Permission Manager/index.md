# Permission Manager

The `PermissionManager` allows developers to easily query whether specific RPCs are allowed or not. It also allows a listener to be added for a list of RPCs so that if there are changes in their permissions, the app will be notified.


## Querying Permission
Using the `PermissionManager`, you can easily know if a specific RPC is allowed or not. For example, if you want to check if the `Show` RPC is allowed you can use the `isRPCAllowed` method:


```java
boolean allowed = sdlManager.getPermissionManager().isRPCAllowed(FunctionID.SHOW);
```

!!! NOTE
Some RPCs are allowed in specific hmi levels but not allowed in others.
!!!

## Querying Permission Parameters
Some RPCs have parameters. For example, `GetVehicleData` has parameters like `speed`, `rpm`, and `airbagStatus`. The developer may need to know not only whether `GetVehicleData` is allowed but also if a specific parameter in that RPC is allowed. For that case the `isPermissionParameterAllowed` method can be used to tell if the RPC and the parameter are both allowed:

```java
boolean allowed = sdlManager.getPermissionManager().isPermissionParameterAllowed(FunctionID.GET_VEHICLE_DATA, GetVehicleData.KEY_RPM);
```

## Querying Multiple Permissions at Once
In some cases, developers may need to know whether multiple permissions (or permission parameters) are allowed and perform a specific action based on the result. The `PermissionManager` has a convenience method that does that. For example, if the developers need to know whether `Show` and `GetVehicleData` RPCs are allowed and also make sure that `speed` and `rpm` parameters in `GetVehicleData` are allowed, they can use `getGroupStatusOfPermissions` method to do that. First, a list of `PermissionElement`s should be created. Each `PermissionElement` in the list holds the RPC that we want to check the permission for and a list of optional parameters for that permission:


```java
List<PermissionElement> permissionElements = new ArrayList<>();
permissionElements.add(new PermissionElement(FunctionID.SHOW, null));
permissionElements.add(new PermissionElement(FunctionID.GET_VEHICLE_DATA, Arrays.asList(GetVehicleData.KEY_RPM, GetVehicleData.KEY_SPEED)));

int groupStatus = sdlManager.getPermissionManager().getGroupStatusOfPermissions(permissionElements);

switch (groupStatus) {
    case PermissionManager.PERMISSION_GROUP_STATUS_ALLOWED:
        // Every permission in the group is currently allowed
        break;
    case PermissionManager.PERMISSION_GROUP_STATUS_DISALLOWED:
        // Every permission in the group is currently disallowed
        break;
    case PermissionManager.PERMISSION_GROUP_STATUS_MIXED:
        // Some permissions in the group are allowed and some disallowed
        break;
    case PermissionManager.PERMISSION_GROUP_STATUS_UNKNOWN:
        // The current status of the group is unknown
        break;
}
```

The previous snippet will give a quick generic status for all permissions together. However, if developers want to get a more detailed result about the status of every permission or parameter in the group, they can use `getStatusOfPermissions` method:

```java
List<PermissionElement> permissionElements = new ArrayList<>();
permissionElements.add(new PermissionElement(FunctionID.SHOW, null));
permissionElements.add(new PermissionElement(FunctionID.GET_VEHICLE_DATA, Arrays.asList(GetVehicleData.KEY_RPM, GetVehicleData.KEY_AIRBAG_STATUS)));

Map<FunctionID, PermissionStatus> status = sdlManager.getPermissionManager().getStatusOfPermissions(permissionElements);

if (status.get(FunctionID.GET_VEHICLE_DATA).getIsRPCAllowed()){
    // GetVehicleData RPC is allowed
}

if (status.get(FunctionID.GET_VEHICLE_DATA).getAllowedParameters().get(GetVehicleData.KEY_RPM)){
    // rpm parameter in GetVehicleData RPC is allowed
}
```

## Adding Permissions Change Listener
In some cases, the app may need to be notified when there is a change in some permissions. Developers can use the `PermissionManager` to add a listener that will be called when the specified permissions change. The listener can be called either when there is any change or only when all permissions become allowed. That can be determined by the `PermissionGroupType` value that is passed to the `AddListener` method:


| Permission Group Type | Description |
| --------- | ----- |
| PERMISSION_GROUP_TYPE_ALL_ALLOWED | Be notified when all of the permissions in the group are allowed, or when they all stop being allowed in some sense, that is, when they were all allowed, and now they are not. |
| PERMISSION_GROUP_TYPE_ANY | Be notified when any change in availability occurs among the group. |


For example, to setup a listener that will be called when there is any update to `Show` or `GetVehicleData` permissions or `rpm`, `airbagStatus` parameter permissions in the `GetVehicleData` RPC, you can use the following code snippet:

```java
List<PermissionElement> permissionElements = new ArrayList<>();
permissionElements.add(new PermissionElement(FunctionID.SHOW, null));
permissionElements.add(new PermissionElement(FunctionID.GET_VEHICLE_DATA, Arrays.asList(GetVehicleData.KEY_RPM, GetVehicleData.KEY_AIRBAG_STATUS)));


UUID listenerId = sdlManager.getPermissionManager().addListener(permissionElements, PermissionManager.PERMISSION_GROUP_TYPE_ANY, new OnPermissionChangeListener() {
    @Override
    public void onPermissionsChange(@NonNull Map<FunctionID, PermissionStatus> allowedPermissions, @NonNull int permissionGroupStatus) {
        if (allowedPermissions.get(FunctionID.GET_VEHICLE_DATA).getIsRPCAllowed()) {
            // GetVehicleData RPC is allowed
        }

        if (allowedPermissions.get(FunctionID.GET_VEHICLE_DATA).getAllowedParameters().get(GetVehicleData.KEY_RPM)){
            // rpm parameter in GetVehicleData RPC is allowed
        }
    }
});
```

!!! NOTE
Don't forget to remove the listener using the `removeListener` method when you are done with it.
!!!
