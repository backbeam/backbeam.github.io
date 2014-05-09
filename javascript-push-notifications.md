# Push notifications

In Backbeam you don't send push notifications directly to certain devices. Instead, you subscribe devices to channels and then you send notifications to those channels. Backbeam takes care about what devices are in which channels.

To subscribe a device to a channel you need to know its device token and the gateway. At this moment two gateways are supported: apn (Apple Push Notifications for iOS devices) and gcm (Google Cloud Messaging for Android devices).

For iOS devices the device token is a string with the base64 representation of the `NSData` object you receive in the `application:didRegisterForRemoteNotificationsWithDeviceToken:` method in the `AppDelegate`. For Android devices the device token is the `registration_id` returned in the intent of type `com.google.android.c2dm.intent.REGISTRATION`.

## Subscribing and unsubscribing devices

Once you know the device token you can subscribe that device to one or more channels easily.

```javascript
var device = 'abcdefabcdefabcdefabcdefabcdef'
backbeam.subscribeDeviceToChannels('gcm', device, 'channel1', 'channel2', function(err) {
  // the device is now subscribed if no error ocurred
})
```

Unsubscribing a device from one or more channels is very similar:

```javascript
var device = 'abcdefabcdefabcdefabcdefabcdef'
backbeam.unsubscribeDeviceFromChannels('gcm', device, 'channel1', 'channel2', function(err) {
  // the device is now unsubscribed from those channels if no error ocurred
})
```

You can also unsubscribe a device from all the channels

```javascript
var device = 'abcdefabcdefabcdefabcdefabcdef'
backbeam.unsubscribeDeviceFromAllChannels('gcm', device, function(err) {
  // the device is now unsubscribed from all channels if no error ocurred
})
```

And finally you can query the channels that a device is subscribed to:

```javascript
var device = 'abcdefabcdefabcdefabcdefabcdef'
backbeam.subscribedChannels('gcm', device, function(err, channels) {
  // if no error ocurred you will get all the channels in the `channels` array
})
```

## Sending push notifications

At any moment you can send push notifications to a channel

```javascript
var options = {
  apn_alert: 'Hello from node.js',
  apn_badge: 10,
  apn_payload_custom_key: 'custom value'
}
backbeam.sendPushNotification('channel1', options, function(err) {
  // notification sent to every device subscribed to that channel
})
```

The options object can have the following parameters:

* `apn_alert`
The text to be shown in the iOS push notifications
* `apn_badge`
An optional number that will be added to the app icon for iOS devices
* `apn_sound`
An optional name of a sound file to be played when a notification arrives to a iOS device
* `apn_payload_*`
You can include additional information adding keys with this prefix for iOS devices.
* `gcm_collapse_key`
It is an arbitrary string that is used to collapse a group of like messages when the device is offline
* `gcm_delay_while_idle`
A boolean used to request messages not be delivered until the device becomes active
* `gcm_data_*`
You can include additional information adding keys with this prefix for Android devices.
