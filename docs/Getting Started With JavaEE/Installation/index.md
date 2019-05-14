## Installation

### Introduction

Each SDL JavaEE library release is published to [Github](https://github.com/smartdevicelink/sdl_java_suite). By building and importing the library JAR file to the project, developers can compile with the latest SDL JavaEE release. In this guide we exclusively use IntelliJ to compile and build the project.

### Building the JavaEE Library JAR    
To build the library JAR from the source code, first clone the [SDL Java Suite](https://github.com/smartdevicelink/sdl_java_suite) repository then, simply call:

```
gradle build
```

from within the JavaEE directory and a JAR should be generated in the build/libs folder.

### Creating a New SDL Project
* [Download Glassfish 5.0.0 Full Platform](https://javaee.github.io/glassfish/download)
* Start a new IntelliJ Project (Menu -> New -> Project)
* Select Java Enterprise as project type.
* Ensure Java EE 8 is the version used.
* Set Application Server to the directory of Glassfish 5.0.0 download.
* Check the following: (they should all be using the libraries from Glassfish)
    * Glassfish 5.0.0 - EJB
    * Glassfish 5.0.0 - WebSocket
    * Glassfish 5.0.0 - Web Application
* Give the project a name.
* Once the project is created, add the SDL Java JAR library
  (Right-click project -> Open Module Settings -> Libraries -> +)
* A problem may appear concerning the exploded war not having the library. Go to artifacts.
  Add SDL Java to the war exploded artifact (IntelliJ has an auto fix for it).
  Apply and OK.
* You can now start creating your SDL enabled application. If you already have source code to start with, you can copy it into the new project along with any jars and assets.

!!! NOTE
Glassfish 5.0.0 only works on JDK 8 and lower.
!!!