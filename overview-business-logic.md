# Where to write the business logic

Imagine you are writing a game and when the player kills an enemy you want to do two things: remove the enemy object and modify the player object to increase its score. That's your *business logic* in this use case.

With Backbeam you can write your business logic in the client-side and in the server-side. When you write client-side logic you get full access from your SDK to the database, and you do all the database operations from the client code. For example from your Objective-C code on iOS or from the Java code on Android.

Since using client-side business logic gets full access to the database all the HTTP requests to the server are secured with API keys and strong security mechanisms.

When you write server-side logic you write JavaScript inside web controllers that are invoked from your client-side code. These web controllers only do what you want to do.

We support both ways to write an application because both have pros and cons. So you are free to pick the way that best fits your needs and knoweldge.

These are some things to have in mind:

*  Client-side logic is easier for mobile developers because you don't have to write server-side code in JavaScript. You just need to write code in your usual development environment.
*  The server-side logic can be updated at any moment without releasing new versions of your application. Imagine you want to do something additional in your business logic such as saving statistics, sending emails, sending push notificaitons, etc. You can just update the server-side code and you won't have to change your client-side code. So all the installed applications are *updated* with the new business logic instantly.
*  The client-side logic requires API keys. The server-side logic don't, so it doesn't have secrets inside the code.
* Server-side logic can be written once and be used in any platform (iOS, Android, web)

## More details about server-side logic

### Objects or JSON

When you write server-side code you will have the option to return *backbeam objects* or return JSON instead. The first option is easier to implement because Backbeam will serialize / deserialize the objects for you. So for example you can make a query in the server-side, Backbeam will serialize it the client-side code will receive it like objects. You don't have the need to parse anything.

If you prefer to have full control about how the data is serialized and received from the backend you can always just return JSON from your controllers.

### Users authentication

A great thing about server-side logic is that the SDK and the backend are able to synchronize the session status automatically. So if you do some authentication stuff in a web controller the SDK will know it and the *currentUser* object in your client-side code will be set automatically. And you don't have to write any additional code to persist or update that information.
