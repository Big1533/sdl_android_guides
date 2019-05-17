## Audio Streaming

!!! IMPORTANT
This feature is only available on Android apps. Currently, JavaSE (embedded) and JavaEE (cloud) apps don't support that.
!!!

Navigation apps are allowed to stream raw audio to be played by the head unit. The audio received this way is played immediately, and the current audio source will be attenuated. The raw audio has to be played with the following parameters:

* **Format**: PCM
* **Sample Rate**: 16k
* **Number of Channels**: 1
* **Bits Per Second (BPS)**: 16 bits per sample / 2 bytes per sample

You can now also push `mp3` files using the `AudioStreamingManager`, which is accessed through the `SdlManager`.


!!! Note
For streaming consistent audio, such as music, use a normal A2DP stream and not this method.
!!!

#### Streaming Audio 

To stream audio, we call `sdlManager.getAudioStreamManager().start()` which will start the manager. When that callback returns successful, you call `sdlManager.getAudioStreamManager().startAudioStream()`. When the callback for that is successful, you can push the audio source using `sdlManager.getAudioStreamManager().pushAudioSource()`. Below is an example of playing an `mp3` file that we have in our resource directory:

```java
if (sdlManager.getAudioStreamManager() != null) {
    Log.i(TAG, "Trying to start audio streaming");
    sdlManager.getAudioStreamManager().start(new CompletionListener() {
        @Override
        public void onComplete(boolean success) {
            if (success) {
                sdlManager.getAudioStreamManager().startAudioStream(false, new CompletionListener() {
                    @Override
                    public void onComplete(boolean success) {
                        if (success) {
                            Resources resources = getApplicationContext().getResources();
                            int resourceId = R.raw.exampleMp3;
                            Uri uri = new Uri.Builder()
                            .scheme(ContentResolver.SCHEME_ANDROID_RESOURCE)
                            .authority(resources.getResourcePackageName(resourceId))
                            .appendPath(resources.getResourceTypeName(resourceId))
                            .appendPath(resources.getResourceEntryName(resourceId))
                            .build();
                            sdlManager.getAudioStreamManager().pushAudioSource(uri, new CompletionListener() {
                                @Override
                                public void onComplete(boolean success) {
                                    if (success) {
                                        Log.i(TAG, "Audio file played successfully!");
                                    } else {
                                        Log.i(TAG, "Audio file failed to play!");
                                    }
                                }
                            });
                        } else {
                            Log.d(TAG, "Audio stream failed to start!");
                        }
                    }
                });
            } else {
                Log.i(TAG, "Failed to start audio streaming manager");
            }
        }
    });
}
```

#### Stopping the Audio Stream

When the stream is complete, or you receive HMI_NONE, you should stop the stream by calling:

```java
sdlManager.getAudioStreamManager().stopAudioStream(new CompletionListener() {
    @Override
    public void onComplete(boolean success) {

    }
});
```
