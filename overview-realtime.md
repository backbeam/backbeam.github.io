# Realtime

## How it works

Backbeam supports a realtime API. It is based on [socket.io](http://socket.io) and it uses websockets under the hood.

## Why a realtime API

A realtime API gives you the opportunity to implement realtime web and mobile applications. You can implement **chat / instant messaging applications and collaborative tools** for example. For example if you are developing a productivity application don't wait for the user to refresh the app/page: notify the user immediatly about any change done by his teammates.

But there is more. You can implement **multiplayer games** or use it on any other application. Engage your users having instant feedback about what's happening.

## Availability

We have used the realtime API extensively and we are very glad about its reliability, but you have to be aware that in some circumstances it can be unavailable. For example:

* For realtime web applications very old browsers could not support it.
* Some network environments prevent it to work. For example some proxies do not implement the HTTP protocol properly and do not support websockets.

So you shlould develop your application having in mind that the service could not be available on some circumstances.

## Security considerations

For technical reasons the information exchanged using the realtime API is not signed using API keys (the API keys would be visible in a web browser for example). So if you require very high security levels regarthing the information you exchange then use the realtime API as a notification system and when a notification is processed implement a check system using the regular Backbeam API (i.e. making a query to the database to double check the information received).
