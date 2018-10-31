# Adding the Lock Screen

In order for your SDL application to be certified with most OEMs you will be required to implement a lock screen on the mobile device. The lock screen will disable user interactions with the application on the mobile device while they are using the head-unit to control application functionality. OEMs may choose to send their logo for your app's lock screen to use; the `LockScreenManager` takes care of this automatically using the default layout.

!!! NOTE
This guide assumes that you have an SDL Service implemented as defined in the [Getting Started](/guides/android/getting-started/) guide.
!!!

There is a manager called the `LockScreenManager` that is accessed through the `SdlManager` that handles much of the logic for you. If you have implemented the `SdlManager` and have defined the `SDLLockScreenActivity` in your manifest but have not defined any lock screen configuration, you are already have a working default configuration. This guide will go over specific configurations you are able to implement using the `LockScreenManager` functionality.

## Lock Screen Activity

You must declare the `SDLLockScreenActivity` in your manifest. To do so, simply add the following to your app's `AndroidManifest.xml` if you have not already done so:

```xml
<activity android:name="com.smartdevicelink.managers.lockscreen.SDLLockScreenActivity"
                  android:launchMode="singleTop"/>
```

!!! MUST
This manifest entry must be added for the lock screen feature to work.
!!!

## Configurations

The default configurations should work for most app developers and is simple to get up and and running. However, it is easy to perform deeper configurations to the lock screen for your app. Below are the options that are available to customize your lock screen which builds on top of the logic already implemented in the `LockScreenManager`.

There is a setter in the `SdlManager.Builder` that allows you to set a `LockScreenConfig` by calling `builder.setLockScreenConfig(lockScreenConfig)`. The following options are available to be configured with the`LockScreenConfig`.

In order to to use these features, create a `LockScreenConfig` object and set it using `SdlManager.Builder` before you build `SdlManager`.

#### Custom Background Color

In your `LockScreenConfig` object, you can set the background color to a color resource that you have defined in your `Colors.xml` file:

```java
lockScreenConfig.setBackgroundColor(resourceColor); // For example, R.color.black
```

#### Custom App Icon

In your `LockScreenConfig` object, you can set the resource location of the drawable icon you would like displayed:

```java
lockScreenConfig.setAppIcon(appIconInt); // For example, R.drawable.lockscreen_icon
```

#### Showing The Device Logo

This sets whether or not to show the connected device's logo on the default lock screen. The logo will come from the connected hardware if set by the manufacturer. When using a Custom View, the custom layout will have to handle the logic to display the device logo or not. The default setting is false, but some OEM partners may require it.

In your `LockScreenConfig` object, you can set the boolean of whether or not you want the device logo shown, if available:

```java
lockScreenConfig.showDeviceLogo(true);
```

#### Setting A Custom Lock Screen View

If you'd rather provide your own layout, it is easy to set. In your `LockScreenConfig` object, you can set the reference to the custom layout to be used for the lock screen. If this is set, the other customizations described above will be ignored:

```java
lockScreenConfig.setCustomView(customViewInt);
```

#### Disabling the Lock Screen Manager:

Please note that a lock screen will likely be required by OEMs. You can disable the `LockScreenManager`, but you will then be required to create your own implementation. This is not recommended as the `LockScreenConfig ` should enable all possible settings while still adhering to most OEM requirements. However, if it  is unavoidable to create one from scratch the `LockScreenManager` can be disabled via the `LockScreenConfig` as follows.

```java
lockScreenConfig.setEnabled(false);
```

!!! NOTE
When the enabled flag is set to `false` all other config options will be ignored.
!!!








