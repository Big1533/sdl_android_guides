# Updating from 4.6 to 4.7











## Video Streaming:

Previously, developers had to make sure that the app was in HMI_FULL before starting the video stream, In 4.7, the developer can start video streaming in `VideoStreamingManager.start()`'s `CompletionListener`. The `VideoStreamingManager` will take care of starting the video when the app becomes ready.

### In 4.6:

```java
@Override
public void onOnHMIStatus(OnHMIStatus notification) {
    if(notification.getHmiLevel().equals(HMILevel.HMI_FULL)){
        if (notification.getFirstRun()) {
            proxy.startRemoteDisplayStream(getApplicationContext(), MyDisplay.class, null, false);
        }
    }

}
```

### In 4.7:

```java
sdlManager.getVideoStreamManager().start(new CompletionListener() {
    @Override
    public void onComplete(boolean success) {
        if (success) {
            sdlManager.getVideoStreamManager().startRemoteDisplayStream(getApplicationContext(), MyDisplay.class, null, false);
        } 
    }
});
```
