# Push notifications

First of all you need to configure your project and your application to be able to send and receive push notifications. You need to get a an API key in the [Google APIs console](https://code.google.com/apis/console/) for the [Google Cloud Messaging service](http://developer.android.com/google/gcm/index.html). Then edit your project configuration and save this project key. Finally edit your `AndroidManifest.xml` with the proper permissions and the following services. Example:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.helloworld"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="16" />

    <!-- minimal required permissions -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:allowBackup="true"
        android:theme="@style/AppTheme" >
        <activity
            android:name=".MainActivity"
            android:label="@string/title_activity_main" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- minimal required services -->
        <receiver
            android:name="io.backbeam.GCMBroadcastReceiver"
            android:permission="com.google.android.c2dm.permission.SEND" >
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
                <category android:name="com.example.helloworld" />
            </intent-filter>
        </receiver>
        <service android:name="io.backbeam.GCMIntentService" />
    </application>

</manifest>
```

Then you need to register your device to be able to receive push notificaitons.

```java
String senderID = "1234567890123"; // Google gives you this id
Backbeam.enableGCM(senderID, new GCMCallback() {

  @Override
  public void unrecoverableError(String error) {
  }

  @Override
  public void serviceNotAvailable() {
  }

  @Override
  public void deviceUnregistered(String registrationId) {
  }

  @Override
  public void deviceRegistered(String registrationId) {
    // Device ready to be subscribed to push notification channels
  }

})
// all methods are optional and you can even pass null (no GCMCallback)
```

In Backbeam you don't send push notifications to a certain device but you send notifications to channels. In the Android SDK you can subscribe a device to one or more channels by using the corresponding methods of the Backbeam class.

## Receiving push notifications

To receive notifications sent by the server or other devices you need to set the "notification handler" as follows:

```java
Backbeam.setPushNotificationHandler(new IntentCallback() {
  public void handleMessage(Intent intent) {
    String value = intent.getStringExtra("an-android-key");
    System.out.println("message! value="+value);
  }
  public void failure(BackbeamException exception) {
    System.out.println("push notification exception: "+exception);
  }
});
```

## Subscribing the device to push notifications

If you are using client-side logic (remember to configure your API keys), you can subscribe and unsubscribe the device to any channel directly from your Android code. Let's see how to subscribe the device to a couple of channels:

```java
Backbeam.subscribeToChannels(new OperationCallback() {
  public void success() {
    System.out.println("successfully subscribed");
  }
  public void failure(BackbeamException exception) {
    System.out.println("subscription failure "+exception);
  }
}, "foo-channel", "bar-channel");
```

You can unsubscribe the current device from any channels as follows:

```java
Backbeam.unsubscribeFromChannels(new OperationCallback() {
  public void success() {
    System.out.println("successfully unsubscribed");
  }
  public void failure(BackbeamException exception) {
    System.out.println("subscription failure "+exception);
  }
}, "foo-channel", "bar-channel");
```

If you are using server-side logic you should send the device token (`registrationId`) to a web controller. Then that controller should subscribe the device to the desired channels. You can always access the registrationId using `Backbeam.registrationId()` (if it was requested and stored correctly).

## Sending push notifications

You can send push notifications from your Android code if you are using client-side business logic.

To send a notification to a channel you first create a `PushNotification` object and then you call `Backbeam.sendPushNotificationToChannel()`.

```java
PushNotification notification = new PushNotification();
notification.addAndroidData("an-android-key", "an-android-value");
notification.setAndroidCollapseKey("This is the collapse key!!");
notification.setAndroidDelayWhileIdle(Boolean.FALSE);
Backbeam.sendPushNotificationToChannel(notification, "foo-channel", new OperationCallback() {
  public void success() {
    System.out.println("push sent successfully");
  }
  public void failure(BackbeamException exception) {
    System.out.println("sent failure "+exception);
  }
});
```

If you are using server-side logic then the server (any of your web controllers) should be the one that should send the push notification.
