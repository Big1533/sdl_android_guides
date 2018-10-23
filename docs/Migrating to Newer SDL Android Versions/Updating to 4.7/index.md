### Sending RPCs

There are new method names and locations that mimic previous functionality for sending RPCs. These methods are located in the `SdlManager` and have the new names of `sendRPC`, `sendRPCs`, and `sendSequentialRPCs`.

##### 4.6:

```java
// single RPC
proxy.sendRPCRequest(request);

// muliple RPCs, non-sequential
proxy.sendRequests(rpcs, new OnMultipleRequestListener() {
	//...
});

// multiple RPCs, sequential
proxy.sendSequentialRequests(rpcs, new OnMultipleRequestListener() {
	//...
});
```

In 4.7, we use the `SdlManager` to send the requests.

##### 4.7:

```java
// single RPC
sdlManager.sendRPC(request);

// muliple RPCs, non-sequential
sdlManager.sendRPCs(rpcs, new OnMultipleRequestListener() {
	//...
});

// multiple RPCs, sequential
sdlManager.sendSequentialRPCs(rpcs, new OnMultipleRequestListener() {
	//...
});
```
