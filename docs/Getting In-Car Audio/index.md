## Getting In-Car Audio

Capturing in-car audio allows developers to interact with users via raw audio data provided to them from the car's microphones. In order to gather the raw audio from the vehicle, we must leverage the `PerformAudioPassThru` RPC.

!!! note
PerformAudioPassThru does not support automatic speech cancellation detection, so if this feature is desired, it is up to the developer to implement.
!!!


### Subscribing to AudioPassThru Notifications

Before starting audio capture, the app has to subscribe to `AudioPassThru` notification. SDL provides audio data as fast as it can gather it, and sends it to the developer in chunks. In order to retrieve this audio data, observe the `OnAudioPassThru` notification:

```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_AUDIO_PASS_THRU, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnAudioPassThru onAudioPassThru = (OnAudioPassThru) notification;
        byte[] dataRcvd = onAudioPassThru.getAPTData();
        processAPTData(dataRcvd); // Do something with audio data
    }
});
```

!!! note
This audio data is only the current audio data, so the developer must be in charge of managing previously retrieved audio data.
!!!


### Starting Audio Capture
To initiate audio capture, we must construct a `PerformAudioPassThru` object. The properties we will set in this object's constructor relate to how we wish to gather the audio data from the vehicle we are connected to.

```java
PerformAudioPassThru performAPT = new PerformAudioPassThru();
performAPT.setAudioPassThruDisplayText1("Ask me \"What's the weather?\"");
performAPT.setAudioPassThruDisplayText2("or \"What's 1 + 2?\"");

performAPT.setInitialPrompt(TTSChunkFactory.createSimpleTTSChunks("Ask me What's the weather? or What's 1 plus 2?"));
performAPT.setSamplingRate(SamplingRate._22KHZ);
performAPT.setMaxDuration(7000);
performAPT.setBitsPerSample(BitsPerSample._16_BIT);
performAPT.setAudioType(AudioType.PCM);
performAPT.setMuteAudio(false);

sdlManager.sendRPC(performAPT);
```

!!! note
`AudioPassThru` notification listener should be added before sending `PerformAudioPassThru` request  or else some audio data may be missed. 
!!!


#### Ford HMI
![Ford Audio Pass Thru](assets/Ford_AudioPassThruPrompt.png)

In order to know the currently supported audio capture capabilities of the connected head unit, please refer to the `SystemCapabilityManager`. It can retrieve the `AudioPassThruCapabilities` that the head unit supports.

!!! note
Currently, Ford's SYNC 3 vehicles only support a sampling rates of 16 khz and a bit rate of 16.
!!!


### Ending Audio Capture
`AudioPassThru` is a request that works in a different way when compared to other RPCs. For most RPCs a request is followed by an immediate response that informs the developer whether or not that RPC was successful. This RPC, however, will only send out the response when the Perform Audio Pass Thru is ended.

Audio Capture can be ended in 4 ways:

1. `AudioPassThru` has timed out.

    If the audio passthrough has proceeded longer than the requested timeout duration, Core will end this request and send a `PerformAudioPassThruResponse` with a `Result` of `SUCCESS`. You should expect to handle this audio passthrough as though it was successful.

2. `AudioPassThru` was closed due to user pressing "Cancel".

    If the audio passthrough was displayed, and the user pressed the "Cancel" button, you will receive a `PerformAudioPassThruResponse` with a `Result` of `ABORTED`. You should expect to ignore this audio pass through.

3. `AudioPassThru` was closed due to user pressing "Done".

    If the audio passthrough was displayed, and the user pressed the "Done" button, you will receive a `PerformAudioPassThruResponse` with a `Result` of `SUCCESS`. You should expect to handle this audio passthrough as though it was successful.

4. `AudioPassThru` was ended due to the developer ending the request.

    If the audio passthrough was displayed, but you have established on your own that you no longer need to capture audio data, you can send an `EndAudioPassThru` RPC.

```java
EndAudioPassThru endAPT = new EndAudioPassThru();
sdlManager.sendRPC(endAPT);
```

You will receive an `EndAudioPassThruResponse` and a `PerformAudioPassThruResponse` with a `Result` of `SUCCESS`, and should expect to handle this audio passthrough as though it was successful.

### Handling the Response
To process the response that we received from an ended audio capture, we monitor the `PerformAudioPassThruResponse` by adding a listener to the `PerformAudioPassThru` RPC before sending it. If the response has a successful `Result`, all of the audio data for the passthrough has been received and is ready for processing.

```java
performAPT.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        Result result = response.getResultCode();

        if(result.equals(Result.SUCCESS)){
            // We can use the data
        }else{
            // Cancel any usage of the data
            Log.e("SdlService", "Audio pass thru attempt failed.");
        }
    }
});
```
