# JavaScript SDK

This documentation is valid for both the server-side code (libs and controllers in a backbeam project) and for the backbeam.js SDK. The backbeam.js SDK is intented to be used in embedded mobile web applications (i.e. applications based on a webview embedded in a native application) or as a Node.js package. This JavaScript SDK has also been tested in Titanium Appcelerator.

Do not include your API keys in any place that can be visible easily. If you are doing a hosted web application take a look at the [micro JavaScript SDK](https://backbeam.io/article/introducing-the-new-micro-javascript-sdk).

The source code of this SDK is hosted in [a repository in GitHub](https://github.com/backbeam/backbeamjs).

## Dependencies

When using the JavaScript SDK inside a webview or a web browser a few functions need some libraries in order to do some cryptography. They are included in the git repository: hmac-sha1.js and enc-base64.js. Additionally if you want to use the cache mechanism in your queries you need to include the cache.js file.

The node.js package is available using [npm](https://npmjs.org/package/backbeam). You can install it using `npm install backbeam`.

The server-side code has no dependencies. You can use the backbeam object directly.

## Configure your application

This step is not needed for server-side code. It is also not required if you are using the micro-SDK

```javascript
backbeam.configure({
    project: 'your-project-name', // the subdomain of your project
    env: 'dev', // can be 'dev' or 'pro'
    protocol: 'http', // can be 'http' or 'https'

    // The shared and secret keys are not needed if you only invoke controllers or use the real-time API
    shared: 'your-shared-key',
    secret: 'your-secret-secret',

    // These are only needed if you invoke controllers
    webVersion: 'v1', // it's the web version you want to invoke
    httpAuth: 'xxxx', // it's the authorization password you will see in 'Views and controllers' in your control panel
})
```

## Objects

Most of the time you will read and manipulate the data stored in the database using objects. You will receive objects when making queries and you will create and update objects to manipulate the data in the database. You will access its fields and some other metainformation. These are the most important methods in these objects:

* `id()`. Returns a string containing the unique object identifier
* `createdAt()`. Returns a Date object indicating when the user was created
* `updatedAt()`. Returns a Date object indicating when the user was updated for the last time
* `get(field)`. Returns the value of the given field
* `set(field, val)`. Sets the value of the given field. The new value is not synched with the backend until you perform a `save()` operation
* `add(field, obj)`. Adds an object to a relationship. For example you could do event.add('attendees', user) if you have an 'event' entitity with a relationship 'to many' called 'attendees'. The relationship is synched when you perform a `save()` operation
* `rem(field, obj)`. The opposite of the add() method.

## Join results

Many times you will perform joins in your BQL queries and other operations. If the join is made in a relationship "to one" you can just use `object.get('fieldName')` to get the value of the relationship. Let's see an example of a BQL query with a join "to one" (you will learn more about queries in the next section):

```javascript
backbeam.select('event').query('join place').fetch(10, 0, function(err, objects, totalCount, fromCache) {
    var event = objects[0] // pick an event (in real code check the error and the array length)
    var place = event.get('place') // here you have the joined object
})
```

You can also make joins in the "to many" side of a relationship. In that case you can fetch a number of objects in the relationship and you always can get the total number of objects in the relationship.

```javascript
backbeam.select('place').query('join last 10 events').fetch(10, 0, function(err, objects, totalCount, fromCache) {
    var place = objects[0] // pick a place (in real code check the error and the array length)
    var join = place.get('events') // here you have a special object with two attributes

    var events = join.result // this is an array of `event` objects
    var count = join.count // total count of objects in the relationship

    // For example you can iterate the joined objects of the picked place
    for (var i=0; i<events.length; i++) {
        var event = events[i]
        var name = event.get('name')
        // ...
    }
})
```

If you are only interested in the number of objects in a relationship you can just query using for example join events. Check the BQL documentation for further information.

## Special objects

There are a couple of special objects. When using fields of type location, their values are encapsulated in objects of type backbeam.Location. This object has the following attributes:

* `addr` The human readable address (can contain the country, street,...)
* `lat` The latitude
* `lon` The longitude
* `alt` The altitude. Always optional

You can create a location object like this:

```javascript
new backbeam.Location('Barcelona', 41.403611, 2.174444) // you can add an optional last parameter for the altitude
```

For fields of type day there is also a special object type: `backbeam.Day`. This object is timezone-agnostic. That's why the native `Date` object is not used for day fields. There are multiple ways to create a backbeam.Day object

```javascript
var today = new backbeam.Day()
var dayFromDate = new backbeam.Day(someJSDateObject) // uses Date.getFullYear(), Date.getMonth()+1 and Date.getDate() internally
var otherDay = new backbeam.Day(2013, 8, 31)
```

A backbeam.Day has the following attributes:

* `year` The year
* `month` The month (Jaunuary == 1)
* `day` The day of the month

## Users authentication

One of the functionalities of the SDK is to remember the authenticated user between sessions of the application. You can always access the current authenticated user using `backbeam.currentUser()` and you can logout the user by calling `backbeam.logout()`. In further sections you will see how to authenticate your user, but these methods apply to all of the authentication mechanisms.

If the user has been authenticated using an external provider (Twitter, Facebook, etc.) you can access information of that user from the external identity providers by using some of these methods:

* user.getTwitterData(key)
* user.getFacebookData(key)
* user.getGooglePlusData(key)
* user.getLinkedInData(key)
* user.getGitHubData(key)

For example you can access the id, name or link of a user authenticated with Facebook. If the user has been signed up with Twitter you can access the id, name, screen_name or image and finally if a user has been signed up with Google+ you can access the id, name or image properties.

Users can have multiple authentication mechanisms. If a user is already authenticated and you authenticate it agaist Twitter, Facebook or Google+ authentication then the user is linked to these new providers. No new account is created, the already existing user is liked to those providers. If that's not the behaviour you are expecting just call `backbeam.logout()` first.

## Caching

Queries and controllers invocations can be cached. In order to use this caching mechanism you need to call the policy method in the query object:

```javascript
backbeam.select('event').query('join place').policy('local or remote').fetch(10, 0, function(err, objects, totalCount, fromCache) {
  // do something
})
```

The possible cache policies are:

* `"local"`
The query result is read from the cache. If no value is present an error is passed to the callback function
remote
The cache is not read. The result is fetched only from the remote server
* `"local and remote"`
The query result is read from the cache. Then the result is fetched from the remote server. If the value is not present in the cache, the callback function is only called once. If a value is found in the cache the callback function will be called twice.
* `"local or remote"`
The query result is read from the cache. If a value is found the callback function is called and the remote server is not called. If there is not a cached result the server is called and the callback function is called once.

You can always know if a value has been fetched from the cache or not using the fourth parameter of the callback that is a boolean value whose value will be true if the value was read from the cache.

In order to use the cache mechanism you need to configure it in the configure method:

```javascript
backbeam.configure({ ..., cache: { type: 'default' }}
```

At this moment there are two builtin cache implementations. You can use default to use an in-memory cache. You can set the cache type to `"localStorage"` to use a cache implementation that uses the web browser's local storage (supported in most browsers).

You can configure your own cache implementation using:

```javascript
{ type: 'customStorage', impl: new MyCacheImpl()}
```

Where `impl` is an object with the same API than the `JSCache` library.
