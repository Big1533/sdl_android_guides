## Installation

### Introduction

Each [SDL JavaEE](https://github.com/smartdevicelink/sdl_java_suite) library release is published to JCenter. By adding a few lines in their app's gradle script, developers can compile with the latest SDL JavaEE release.

To gain access to the JCenter repository, make sure your app's `build.gradle` file includes the following:

```
repositories {
    google()
    jcenter()
}
```

### Gradle Build

To compile with the a release of SDL JavaEE, include the following line in your app's `build.gradle` file,

```
dependencies {
    implementation 'com.smartdevicelink:sdl_java_ee:{version}'
}
```

and replace `{version}` with the desired release version in format of `x.x.x`. The list of releases can be found [here](https://github.com/smartdevicelink/sdl_java_suite/releases). 

### Examples

To compile release 4.8.0, use the following line:

```
dependencies {
    implementation 'com.smartdevicelink:sdl_java_ee:4.8.0'
}
```

To compile the latest minor release of major version 4, use:

```
dependencies {
    implementation 'com.smartdevicelink:sdl_java_ee:4.+'
}
```
