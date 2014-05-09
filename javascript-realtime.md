# Realtime

## Enabling the realtime

The realtime API is available when the SDK runs in the browser. The server-side implementation is limited by design and can only send realtime events but it cannot receive events.

To enable the realtime API you only need to call the `backbeam.enableRealTime()` method.

## Subscribing to real time events

Send and receive realtime events is very easy. Here is an example of how to subscribe the device to a certain type of events and how to listen to the same type of events:

```javascript
// subsribe to a kind of realtime event
backbeam.subscribeToRealTimeEvents('my-event-name', function(eventName, data) {
    console.log('received', data)
})

// send relatime events
backbeam.sendRealTimeEvent('my-event-name', {say:text})
```

## Listening to real time connection changes

You can also subscribe to the status of the realtime connection. You can listen to one or more event types

```javascript
backbeam.subscribeToRealTimeConnectionEvents({
    connect: function() {
            console.log('connected')
    },
    disconnect: function() {
            console.log('disconnect')
    },
    connecting: function() {
            console.log('connecting')
    },
    connectFailed: function() {
            console.log('connectFailed')
    }
})
```
