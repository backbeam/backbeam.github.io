# Push notifications

With Backbeam you can send push notifications easily. You will subscribe devices to channels and then you will send push notifications to those channels. This is an easy way to group devices and send multiple notifications to any group. For example if you are implementing a chat application you will want to subscribe all the devices in a chat room to one common channel and then with one method call you can send push notifications to all devices subscribed to that channel.

Even if you are writing a chat application that do not support group chats you are still interested in send push notifications easily to many devices at the same time. Note that maybe you only want to send a push notification to one *user* but that user could be signed on more than one device (e.g. a smartphone and a tablet).

## Naming channels

Channels are automatically created when a device subscribes to it. There is no manual step on creating channels and your application is not limited in the number of channels it can *create*. Usually these channels will be created dynamically by the users.

For example you can implement an application where the users create new discussion topics and you want all the users participating on the discussion to receive push notifications about activity updates. You will have for example a `topic` entity in your database and you can subscribe the devices to channels named `topic-[topic-identifier]` (e.g. `topic-uDCuumEaMM4e`).

This is a good way of naming channels: usually there will be something already in the database that groups the device around the same interest and you can use a prefix plus the identifier of that object that groups things together to name the channel. This way you are sure that the channel has a unique and meaningful name.

## Supported platforms

At this moment two platforms are supported

### Apple Push Notification service

This is the service with which iOS devices receive push notifications. Recently is also supported by MacOSX Safari to implement push notifications in websites.

To use this service you need to configure the push notifications certificates in the *Configuration* section of your project.

### Google Cloud Messaging service

This is the official by Google for the Android platform.

To use this service you need to configure an API key in the *Configuration* section of your project. You can get an API key in the [Google APIs console](https://code.google.com/apis/console/).

## Considerations

Neither Apple nor Google ensure the delivery of push notifications. For example if the user's device is switched off or has no signad during a long period some push notifications may be lost.

#### Feedback

Is there any other service you want to be implemented in Backbeam? Leave a comment in that case :)
