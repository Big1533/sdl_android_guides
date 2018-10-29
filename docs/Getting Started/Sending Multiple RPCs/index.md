## Sending Multiple RPCs

## Batch Sending RPCs

There are two ways to send multiple requests to the head unit: concurrently and sequentially. Which method you should use depends on the type of RPCs being sent. Concurrently sent requests might finish in a random order and should only be used when none of the requests in the group depend on the response of another, such as when uploading a group of artworks. Sequentially sent requests only send the next request in the group when a response has been received for the previously sent RPC. Requests should be sent sequentially when you need to know the result of a previous request before sending the next, like when sending the several different requests needed to create a menu.

Both methods have optional listeners that are specific to them, the `OnMultipleRequestListener`. This listener will provide additional information than the normal `OnRPCResponseListener`. Its use is shown below.

### Send Requests

`sendRPCs` allows you to easily send an ArrayList of `RPCRequests` easily to the head unit. When you send multiple RPCs concurrently there is no guarantee of the order in which the RPCs will be sent or in which order Core will return responses. The method also comes with its own listener, `OnMultipleRequestListener` that will provide you with updates as the sending progresses, errors that may arise, and let you know when the sending is finished. Below is a sample call:

```java
List<RPCRequest> rpcs = new ArrayList<>();

// rpc 1
SubscribeButton subscribeButtonRequestLeft = new SubscribeButton();
subscribeButtonRequestLeft.setButtonName(ButtonName.SEEKLEFT);
rpcs.add(subscribeButtonRequestLeft);

// rpc 2
SubscribeButton subscribeButtonRequestRight = new SubscribeButton();
subscribeButtonRequestRight.setButtonName(ButtonName.SEEKRIGHT);
rpcs.add(subscribeButtonRequestRight);


sdlManager.sendRPCs(rpcs, new OnMultipleRequestListener() {
	@Override
	public void onUpdate(int remainingRequests) {

	}

	@Override
	public void onFinished() {

	}

	@Override
	public void onResponse(int correlationId, RPCResponse response) {

	}

	@Override
	public void onError(int correlationId, RPCResponse response) {

	}
});

```

### Send Sequential Requests

As you may have guessed, this method is called similarly to `sendRPCs` but sends the requests synchronously, guaranteeing order. It is important to note that you want to build your array with the items that you want to send first, first. This is particularly useful for RPCs that are dependent upon other ones, such as a `performInteraction` needing a `createInteractionChoiceSet`'s id. 

This method call is exactly the same as above, except for the method name being `sendSequentialRPCs`. For your convenience, the listener is also the same and performs similarly. 

```java

List<RPCRequest> rpcs = new ArrayList<>();

// rpc 1
SubscribeButton subscribeButtonRequestLeft = new SubscribeButton();
subscribeButtonRequestLeft.setButtonName(ButtonName.SEEKLEFT);
rpcs.add(subscribeButtonRequestLeft);

// rpc 2
SubscribeButton subscribeButtonRequestRight = new SubscribeButton();
subscribeButtonRequestRight.setButtonName(ButtonName.SEEKRIGHT);
rpcs.add(subscribeButtonRequestRight);

sdlManager.sendSequentialRPCs(rpcs, new OnMultipleRequestListener() {
	@Override
	public void onUpdate(int remainingRequests) {
	
	}

	@Override
	public void onFinished() {
	
	}

	@Override
	public void onResponse(int correlationId, RPCResponse response) {
	
	}

	@Override
	public void onError(int correlationId, Result resultCode, String info) {
	
	}
});

```

