# Setting Security Level for Multiplexing

When connecting to Core via Multiplex Bluetooth transport, your SDL app will use a Router Service housed within your app or another SDL enabled app.

To help ensure the validility of the Router Service, you can select the security level explicity when you create your Multiplex Bluetooth transport in your app's SdlService:

```java
int securityLevel = FLAG_MULTI_SECURITY_MED;

BaseTransport transport = MultiplexTransportConfig(context, appId, securityLevel);
```

If you create the transport without specifying the security level, it will be set to `FLAG_MULTI_SECURITY_MED` by default.

## Security Levels

Security Flag   | Meaning
------------|------------------------------------------------------------
`FLAG_MULTI_SECURITY_OFF`       | Multiplexing security turned off. All router services are trusted.
`FLAG_MULTI_SECURITY_LOW`  | Multiplexing security will be minimal. Only trusted router services will be used. Trusted router list will be obtained from server. List will be refreshed every 20 days or during next connection session if an SDL enabled app has been installed or uninstalled. 
`FLAG_MULTI_SECURITY_MED`     | Multiplexing security will be on at a normal level. Only trusted router services will be used. Trusted router list will be obtained from server. List will be refreshed every 7 days or during next connection session if an SDL enabled app has been installed or uninstalled.
`FLAG_MULTI_SECURITY_HIGH`	| Multiplexing security will be very strict. Only trusted router services installed from trusted app stores will be used. Trusted router list will be obtained from server. List will be refreshed every 7 days or during next connection session if an SDL enabled app has been installed or uninstalled.

## Applying  to the Trusted Router Service List
To get your app onto the Trusted Router Service List, please contact the SDL Android channel on [Slack](http://slack.smartdevicelink.com) or email [sdladmin@livio.io](mailto:sdladmin@livio.io). We'll need to test and review your app before approving it for the trusted list.
