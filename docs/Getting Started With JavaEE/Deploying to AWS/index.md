## Migrating an SDL Cloud App to JavaEE
#### Updated: April 9, 2019
  
### Table of Contents
- [Project Setup Method 1: New Project](#project-setup-method-1)
- [Migrating your App to the New Project](#migrating-your-app)
- [Project Setup Method 2: Modify Existing Project](#project-setup-method-2)
- [Adding EJB and Websockets](#adding-ejb)
- [SDL Java Changes](#sdl-java-changes)
- [Deploying your JavaEE App on EC2](#deploying-your-javaee-app)
- [Limitations and Issues](#limitations)
- [Useful Information and Commands](#useful-information-and-commands)

### Intro
This document assumes IntelliJ is used as the IDE. There are two ways to set up the project for JavaEE. Method 1 creates a brand new project, and Method 2 modifies an existing one. Pick only one and proceed through all the steps below.

### Project Setup Method 1: New Project
* [Download Glassfish 5.0.0 Full Platform](https://javaee.github.io/glassfish/download)
* Start a new IntelliJ Project (Menu -> New -> Project)
* Ensure Java EE 8 is the version used.
* Set Application Server to the directory of Glassfish 5.0.0 download.
* Select Java Enterprise.
* Check the following: (they should all be using the libraries from Glassfish)
    * Glassfish 5.0.0 - EJB
    * Glassfish 5.0.0 - WebSocket
    * Glassfish 5.0.0 - Web Application
* Give the project a name.
* Once the project is created, add the SDL Java jar library
  (Right-click project -> Open Module Settings -> Libraries -> +)
* A problem may appear concerning the exploded war not having the library. Go to artifacts.
  Add SDL Java to the war exploded artifact (IntelliJ has an auto fix for it).
  Apply and OK.
* You can now start creating your SDL enabled application. If you already have source code to start with, you can copy it into the new project along with any jars and assets.

### Project Setup Method 2: Modify Existing Project
* [Download Glassfish 5.0.0 Full Platform](https://javaee.github.io/glassfish/download)
* Open your project in IntelliJ
* Right-click project -> Open Module Settings -> Modules
* Modules -> Your Module -> Dependencies -> + -> Library
* Add Selected: Glassfish 5.0.0
    * Glassfish 5.0.0 - EJB
    * Glassfish 5.0.0 - WebSocket
    * Glassfish 5.0.0 - Web Application
* Apply
* Inside your existing module, add a web module (+ -> Web)
* Create Artifact and apply fix to missing libraries in artifact. Apply and OK.
* Inside your existing module, also add an EJB module (+ -> EJB). Do not address missing jars.
* Apply and OK.
* Add the SDL Java jar if it doesn’t exist.
* Apply and OK.

### Adding EJB and Websockets
Create a new package where all the JavaEE-specific code will go. 

The SDL Java library comes with a CustomTransport class which takes the role of sending messages between incoming sdl_core connections and your SDL application. You need to pass that class to the SDLManager builder to make the SDL Java library aware that you want to use your JavaEE websocket server as the trasnport.

Create a Java class in the new package which will be the SDLSessionBean class. This class utilizes the CustomTransport class and EJB JavaEE API which will make it the entry point of your app when a connection is made. It will open up a websocket server at `/` and create stateful beans, where the bean represents the logic of your cloud app. Every new connection to this endpoint creates a new bean containing your app logic, allowing for load balancing across all the instances of your app that were automatically created. 

```java
import com.smartdevicelink.transport.CustomTransport;
import javax.ejb.Stateful;
import javax.websocket.*;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.nio.ByteBuffer;

@ServerEndpoint("/")
@Stateful(name = "SDLSessionEJB")
public class SDLSessionBean {

    CustomTransport websocket;

    public class WebSocketEE extends CustomTransport {
        Session session;
        public WebSocketEE(String address, Session session) {
            super(address);
            this.session = session;
        }
        public void onWrite(byte[] bytes, int i, int i1) {
            try {
                session.getBasicRemote().sendBinary(ByteBuffer.wrap(bytes));
            }
            catch (IOException e) {

            }
        }
    }
    
    @OnOpen
    public void onOpen (Session session) {
        websocket = new WebSocketEE("http://localhost", session) {};
        //TODO: pass your CustomTransport instance to your SDL app here
    }

    @OnMessage
    public void onMessage (ByteBuffer message, Session session) {
        websocket.onByteBufferReceived(message); //received message from core
    }
}
```

Unfortunately, [there's no way to get a client's IP address using the standard API](https://stackoverflow.com/a/23025059), so localhost is passed to the CustomTransport for now as the transport address (this is only used locally in the library so it is not necessary). 

The SDLSessionBean class’s @OnOpen method is where you will start your app, and should call your entry of your application and invoke whatever is needed to start it. You need to pass the instantiated CustomTransport object to your application so that the connection can be passed into the SdlManager.

The SdlManager will need you to create a CustomTransportConfig, pass in the CustomTransport instance from the SDLSessionBean instance, then set the SdlManager Builder’s transport type to that config. This will set your transport type into CUSTOM mode and will use your CustomTransport instance to handle the read and write operations.

```java
// Set transport config. builder is a SdlManager.Builder
builder.setTransportType(new CustomTransportConfig(websocket));
```

##### Add a new artifact:

* Right-click project -> Open Module Settings -> Artifacts -> + ->
  Web Application: Archive -> for your war: exploded artifact which should already exist
* Create Manifest. Apply + OK.
* Run Build -> Build Artifacts to get a .war file in the /out folder.

### Deploying your JavaEE App on EC2

When you want to run the project outside the IDE, take the war artifact and deploy it using a Payara server. Payara is built on top of Glassfish and is more well-maintained, and it also solves an issue with Glassfish on redeploying a JavaEE Websocket app where no connections can happen the second time. 

Once you get an EC2 machine, log on it and install some libraries:

```
sudo yum install java-1.8.0-openjdk-headless.x86_64 java-1.8.0-openjdk-devel.x86_64 -y
wget -O payara.zip  https://search.maven.org/remotecontent?filepath=fish/payara/distributions/payara/5.184/payara-5.184.zip
jar xvf payara.zip
cd payara5/glassfish/bin/
sudo chmod 755 asadmin
```

There are two ways to start Payara. The first way will start the server without extra configuration, but it is not in a production context. asadmin’s start-domain command is what runs the Payara server. Then, you need to deploy your war application. Modify the command below to point to where your war file is. When it deploys, it will be running on the root path "/" over port 8080. So, your websocket connection from SDL Core should point to the IP of the machine this is running on, port 8080, and nothing more.

```
./asadmin start-domain
./asadmin deploy --contextroot / ~/my-javaee-app.war
```

The second way is in a production context, but if you’re using a small EC2 machine then it will likely fail due to out of memory errors. This is because the server will consume 2 GB of memory by default. To change this, modify the configuration file located here:

```
payara5/glassfish/domains/production/config/domain.xml
```

Find the part of the file where it says:

```
        <jvm-options>-Xmx2g</jvm-options>
        <jvm-options>-Xms2g</jvm-options>
```

“Xmx2g” means to start the server with 2 GB. Change the number to 1 to make it 1 GB, or change “g” to “m” to make it run in MB. Change both lines.


Starting in production mode:

```
./asadmin start-domain production
./asadmin deploy --contextroot / ~/bocks-ee_war.war
```

Run SDL Core and an HMI. If you're serving the HMI over nginx then nginx should be exposed on a different port than 8080, because Payara runs on 8080 by default. Be sure to modify the default policy table's app_policies object so that SDL Core is aware of where your app is:

```json
"8675829": {
    "keep_context": false,
    "steal_focus": false,
    "priority": "NONE",
    "default_hmi": "NONE",
    "groups": ["Base-4"],
    "RequestType": [],
    "RequestSubType": [],
    "hybrid_app_preference": "CLOUD",
    "endpoint": "ws://$LOCAL_IP:8080/",
    "enabled": true,
    "auth_token": "no auth token",
    "cloud_transport_type": "WS",
    "nicknames": ["App1"]
},
```

### Limitations and Issues

[Follow the guidelines located here.](https://www.oracle.com/technetwork/java/restrictions-142267.html)

These are restrictions for what your logic should do in your EJB. Since this guide puts the whole logic of your app in an EJB, you should follow the restrictions specified above. You can still utilize other aspects of JavaEE to get around some of the limitations, but that will not be covered here.

Notable limitations include not starting or managing threads in your app, not reading from or writing to a file directly, not creating or deleting files or directories, not starting websocket connections yourself and not loading native libraries.

Memory usage increases with both redeployments and with many users connecting and disconnecting over time. The Payara server needs to be shut down to reset memory usage. [This is the only post I could find online which had a similar issue.](https://javaee.groups.io/g/glassfish/topic/6181966#89)

When Payara or Glassfish is unable to handle the load, not only does your JavaEE app stop, but the server also stops.

### Useful Information and Commands

Unzipping a jar file: `unzip my_jar.jar`
    
Packaging all items in your directory to a jar file: `jar cf my_jar.jar *`

When your app gets deployed on Payara on the default domain, it shows up in this directory:

```
payara5/glassfish/domains/domain1/applications
```

However, if the app happens to require loading assets from the same directory it originally resided in, that location changes once it is deployed, and is now located here:

```
payara5/glassfish/domains/domain1/config
```

In the PRODUCTION domain, the locations change to this:

```
payara5/glassfish/domains/production/applications
payara5/glassfish/domains/production/config
```

You can start Payara or Glassfish in different contexts, so be aware of how you started the server because it will change where you should move and modify files.

This is a load test ran using a sample app, connections closing after 5 seconds, and using a t3.small (2 GB memory)

| Connections      | Memory % |
| ----------- | ----------- |
| 0     | 24.8      |
| 300   | 25.2      |
| 700   | 25.4      |
| 1000   | 25.6     |
| 1500   | 26.1     |
| 2000   | 26.2     |
| 3000   | 26.2     |
| 4000   | 26.7     |
| 6000   | 26.8     |
| 8000   | 26.3     |
| 10000   | 27.0    |
| 15000   | 30.0    |
| 20000   | 33.5    |
| 25000   | 37.1    |
| 30000   | 39.8    |
| 40000   | crashed |


The test app could not handle more than 20 new connections per second.

Limit of simultaneous connections:          3562 
After applying the max connections config:  7179

[This page shows how to increase your max connection count for an EC2 machine](https://blog.jayway.com/2015/04/13/600k-concurrent-websocket-connections-on-aws-using-node-js/)
[This page shows the settings to tune for a more effective Payara server](https://blog.payara.fish/fine-tuning-payara-server-in-production)

