# Adding the Lock Screen

In order for your SDL application to be certified with most OEMs you will be required to implement a lock screen on the mobile device. The lock screen will disable user interactions with the application while they are using the head-unit to control application functionality. The URL for the lock screen image to display is send by the head-unit.

!!! NOTE
This guide assumes that you have an SDL Service implemented as defined in the [Getting Started](/guides/android/getting-started/) guide.
!!!

## Lock Screen Activity

First, we need to create a new activity that will host our lock screen. In the activity we add a simple [BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html) that listens for `CLOSE_LOCK_SCREEN_ACTION` intents. In the broadcast receiver we will then call [finish()](https://developer.android.com/reference/android/app/Activity.html#finish()) in order to shutdown the activity. We also check for a Bitmap extra, `LOCKSCREEN_BITMAP_EXTRA`, in the intent that started the activity:

```java
public class LockScreenActivity extends Activity {
    public static final String LOCKSCREEN_BITMAP_EXTRA = "LOCKSCREEN_BITMAP_EXTRA";
    public static final String CLOSE_LOCK_SCREEN_ACTION = "CLOSE_LOCK_SCREEN";

    private final BroadcastReceiver closeLockScreenBroadcastReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            finish();
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        registerReceiver(closeLockScreenBroadcastReceiver, new IntentFilter(CLOSE_LOCK_SCREEN_ACTION));

        setContentView(R.layout.activity_lock_screen);

        Intent intent = getIntent();
        ImageView imageView = (ImageView) findViewById(R.id.lockscreen);

        if(intent.hasExtra(LOCKSCREEN_BITMAP_EXTRA)){
            Bitmap lockscreen = (Bitmap) intent.getParcelableExtra(LOCKSCREEN_BITMAP_EXTRA);
            if(lockscreen != null){
                imageView.setImageBitmap(lockscreen);
            }
        }
    }

    @Override
    protected void onDestroy() {
        unregisterReceiver(closeLockScreenBroadcastReceiver);
        super.onDestroy();
    }
}
```

When the activity is created it will generate a layout file to use. We updated this layout to display a default sample image from `res/drawable` in the middle of the screen:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:gravity="center">
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:src="@drawable/sample_lock" <!-- Replace with your own default image -->
        android:id="@+id/lockscreen"/>
</LinearLayout>
```

In our AndroidManifest we need to define the style and launch mode for our lock screen activity. The theme is set to `@android:style/Theme.Black.NoTitleBar.Fullscreen` and the launch mode is set to [singleInstance](https://developer.android.com/guide/topics/manifest/activity-element.html#lmode):

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.livio.hellosdl">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">

        ...

        <activity android:name=".LockScreenActivity"
            android:theme="@android:style/Theme.Black.NoTitleBar.Fullscreen"
            android:launchMode="singleInstance"/>
            
    </application>

</manifest>
```

## Updating SDL Service
To fully implement a lock screen, you'll want to add a LockScreenManager as a member of your Service.

```java
public class SdlService extends Service implements IProxyListenerALM{
	//...
	
	private LockScreenManager lockScreenManager = new LockScreenManager();
	
	//...
```

The LockScreenManager can download and hold a lock screen Bitmap given a URL and a class that implements its callbacks. You can create a class similar to this:

```java
private class LockScreenDownloadedListener implements LockScreenManager.OnLockScreenIconDownloadedListener{

		@Override
		public void onLockScreenIconDownloaded(Bitmap icon) {
			Log.i(TAG, "Lock screen icon downloaded successfully");
		}

		@Override
		public void onLockScreenIconDownloadError(Exception e) {
			Log.e(TAG, "Couldn't download lock screen icon, resorting to default.");
		}
}
```

You should prepare to accept and download the lock screen image from the URL sent by the head-unit in the `onOnSystemRequest` callback. 

```java
public void onOnSystemRequest(OnSystemRequest notification) {
		if(notification.getRequestType().equals(RequestType.LOCK_SCREEN_ICON_URL)){
			if(notification.getUrl() != null && lockScreenManager.getLockScreenIcon() == null){
				lockScreenManager.downloadLockScreenIcon(notification.getUrl(), new LockScreenDownloadedListener());
			}
		}
	}
```

Now that our lock screen activity and manager are setup, all we have to do is handle starting and stopping the activity. In the `onOnLockScreenNotification` we need to trigger the lock screen to be displayed with the HMI Level is set to FULL and when the lock screen status is required. If available, we also need to pass along the Bitmap for the lock screen image. To finish, we need to close the lock screen when the the lock screen status is OFF:

```java
@Override
public void onOnLockScreenNotification(OnLockScreenStatus notification) {
		if(notification.getHMILevel() == HMILevel.HMI_FULL && notification.getShowLockScreen() == LockScreenStatus.REQUIRED) {
			Intent showLockScreenIntent = new Intent(this, LockScreenActivity.class);
			showLockScreenIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
			if(lockScreenManager.getLockScreenIcon() != null){
				showLockScreenIntent.putExtra(LockScreenActivity.LOCKSCREEN_BITMAP_EXTRA, lockScreenManager.getLockScreenIcon());
			}
			startActivity(showLockScreenIntent);
		}else if(notification.getShowLockScreen() == LockScreenStatus.OFF){
			sendBroadcast(new Intent(LockScreenActivity.CLOSE_LOCK_SCREEN_ACTION));
		}
}
```

We also need to close the lock screen when a disconnect happens. We do that by sending another broadcast in the `SdlService.onDestroy` callback:

```java
@Override
public void onDestroy() {
    ...
    sendBroadcast(new Intent(LockScreenActivity.CLOSE_LOCK_SCREEN_ACTION));
    ...
}
```

Now when the HMI fully displays the application, the mobile device will display the lock screen with an image provided by the head-unit.