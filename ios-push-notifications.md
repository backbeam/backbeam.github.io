# Push notifications

First of all you need to configure your project to be able to send push notifications. You will need to generate Apple Push Notification certificates and upload them to backbeam. Check your project configuration.

Then at some time in your application you should ask permission to your user to retrieve a valid token that will be used to send push notifications. This is the standard code for requesting permission (the easiest thing to do is to put this code in the AppDelegate when your application starts):

```objectivec
[[UIApplication sharedApplication] registerForRemoteNotificationTypes:UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound];
```

In your AppDelegate you need to implement the method that is called when the user grants permission and then you pass the token to the Backbeam SDK:

```objectivec
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
  [Backbeam persistDeviceToken:deviceToken]; // pass the device token to the Backbeam SDK
}
```

## Receiving push notifications
In iOS you are notified via your AppDelegate when a push notifications is received. There are different circumstances in which you can receive a push notification. For example the user can tap a push notification alert while the app is not loaded in memory. So the system opens your application and you are notified about the push notification in the following method:

```objectivec
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    // Backbeam configuration...
    // Setup UI...

    // Handle push notificaiton
    NSDictionary *remoteNotif = [launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey];
    if (remoteNotif) {
        // The application was opened by tapping in a push notification
    }
}
```

If the application is loaded in memory and you receive a push notification there are two circumstances: the application was inactive or the application is being used by the user (active):

```objectivec
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
    if ([[UIApplication sharedApplication] applicationState] == UIApplicationStateInactive) {
      // user tapped in a push notification alert while the application was inactive
    } else {
      // Push notification received and the application is in the foreground.
      // In this scenario the system do not provides a notification alert so you should
      // implement your own
    }
}
```

Subscribing the device to push notifications
If you are using client-side logic (remember to configure your API keys), you can subscribe and unsubscribe the device to any channel directly from your Android code. Let's see how to subscribe the device to a couple of channels:

```objectivec
+ (BOOL)subscribeToChannels:(NSArray*)channels success:(SuccessBlock)success failure:(FailureBlock)failure
```

And of course you can unsubscribe the device from one or more channels using

```objectivec
+ (BOOL)unsubscribeFromChannels:(NSArray*)channels success:(SuccessBlock)success failure:(FailureBlock)failure
```

These methods return NO if the device is not registered for push notifications.

Additionally you can unsubscribe a device from all channels with the following method:

```objectivec
- (void)unsubscribeFromAllChannels:(SuccessBlock)success failure:(FailureBlock)failure
```

Or list the channels a device is subscribed to:

```objectivec
+ (void)subscribedChannels:(SuccessArrayBlock)success
                   failure:(FailureBlock)failure
```


If you are using server-side logic you can pass the device token to a web controller and then subscribe/unsubscribe the device to any channel in your server-side code. You can access the device token of the iOS device at any time using [Backbeam deviceToken] if it was requested and persisted correctly.

## Sending push notifications

You can send push notifications with the iOS SDK if you are using client-side business logic.

```objectivec
BBPushNotification *note = [[BBPushNotification alloc] init];
note.iosAlert = @"Alert message";
note.iosBadge = @99; // optional
// Additionally you can send send custom key-pair values with more information
note.iosPayload = [NSDictionary dictionaryWithObjectsAndKeys:@"this is some value", @"some-key", nil];
[Backbeam sendPushNotification:note toChannel:@"foo"];
````

If you are using server-side logic then the server (any of your web controllers) should be the one that should send the push notification.

## Troubleshooting push notifications

If you have problems with push notifications there are a few things you can check. First of all when you upload your push notifications certificates in the control panel you can navigate to the `Logs` section on the left menu and check that you see one or two messages like these:

> Connecting to APN development (gateway.sandbox.push.apple.com)
> Connecting to APN distribution (gateway.push.apple.com)

If you don't see that message something wrong is happening. If your certificates are wrong you will see a message like this:

> Too many SSL errors on APN connection. Check your certificates. (Underlying error: <APN error description>)

If you see the success message(s) (`Connecting to APN ...`) and you are still not receiving the push notifcations the problem could be in your application rather than in the server-side configuration.

First of all check that you are getting a valid token in your `AppDelegate`. Check that the following method is being called:

```objectivec
- (void)application:(UIApplication *)application
    didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
```

If that method is not being called then check if the following method is being called and in that case inspect the returned error:

```objectivec
- (void)application:(UIApplication *)application
    didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
```

If you are getting a valid token and you are passing it to Backbeam but you are still not receiving push notifications you should check the `Logs` section again. Check if you are getting a message like this:

> Unsubscribing device due to feedback service request: TxcvSFra5qgLA8PFCjcSGeCYudymSAhYHfDpOZQ/F5E=

In that case you are compiling the application with a development certificate and you uploaded a distribution certificate or viceversa. So Apple is rejecting the device token.
