## Compiling with Gradle
### Introduction

Each [SDL Android](https://github.com/smartdevicelink/sdl_android) library release is published to JCenter. By adding a few lines in their app's gradle script, developers can compile with the latest SDL Android release.

To gain access to the JCenter repository, make sure your app's `build.gradle` file includes the following:

```
repositories {
    jcenter()
}
```

### Gradle Build

To compile with the a release of SDL Android, include the following line in your app's `build.gradle` file,

```
dependencies {
    compile 'com.smartdevicelink:sdl_android:{version}'
}
```

and replace `{version}` with the desired release version in format of `x.x.x`. The list of releases can be found [here](https://github.com/smartdevicelink/sdl_android/releases). 

### Examples

To compile release 4.3.0, use the following line:

```
dependencies {
    compile 'com.smartdevicelink:sdl_android:4.3.0'
}
```

To compile the latest minor release of major version 4, use:

```
dependencies {
    compile 'com.smartdevicelink:sdl_android:4.+'
}
```
