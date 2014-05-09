# Server side logic

## Invoking web controllers

If you don't want to implement the business logic in your client-side code you can always implement it in the server-side code (web controllers) and invoke the controllers directly. You can read more about the benefits of write server-side logic.

There are two methods to invoke web controllers directly:

### Using requestObjects

If the web controller sends objects then you can use requestObjects and the SDK will parse the response and convert it into backbeam objects. For example

```javascript
backbeam.requestObjects('GET', '/objects', { param: 'value'}, function(err, objects, total) {
  for (var i=0; i<objects.length; i++) {
    console.log('name =', objects[i].get('name'))
  }
})
```

You can pass parameters to the web controller using the third parameter. The web controller for this example could be something like this:

```javascript
backbeam.query('user')
  .query('where some-field = ?', request.query.param)
  .fetch(100, 0, response)
```

If the server did any authentication stuff (signup, login, logout) you will have the backbeam.currentUser() object synchronized.

### Using requestJSON

If the web controller sends JSON then you should use the requestJSON method. It is very similar to requestObjects.

```javascript
backbeam.requestJSON('GET', '/give-me-json', { param: 'value'}, function(err, json) {
  // do something with `json`
})
```

The web controller for this example could be just something like this (or as complicated as you wish!):

```javascript
resposne.json({ message: 'Hello world!' })
```

This method also updates the backbeam.currentUser() variable if the server-side code did some authentication management.

## Server-side code

In your controllers you can access some default objects to access the HTTP request information or to generate and manipulate the HTTP response

### Sending realtime events

From the server side you cannot subscribe/unsubscribe to realtime events but you can send realtime events. It is as easy as follows:

```javascript
backbeam.sendRealTimeEvent('event-name', { key: 'value' })
```

The second argument should only contain key-value pairs whose values are strings.

### The request object

There are some implicit objects in the server side code. One of them is the request object that contains information about the HTTP request

* `request.body` is a collection of key-value pairs containing the submitted params to the server. For example values submited in an HTTP POST form
* `request.query` is a collection of key-value paris containing the params sent in the querystring part of the URL.
* `request.url` contains the requested URL string
* `request.params` contains a collection of key-value paris containing the dynamic path segments of the URL. For example, if you created a controller associated with this pattern: `/item/:itemid` and the `/item/1234` URL is requested, then request.params will contain an objects as follows: `{ itemid: '1234' }`. So you can access the itemid param by using request.params.itemid
request.acceptedLanguages is an array of languages sent by the http user-agent. For example: `['en_US', 'en', 'es']`
* `request.ip` the requester IP address
* `request.headers` the request headers with the header names in lower case. For example you can get the 'Host' request header with request.headers['host'] or request.headers.host
* `request.files` any file sent to the controller in a multipart request. See bellow how to save a file uploaded into backbeam

### Save uploaded objects

To save a file sent to the controller there is a special saveFile method:

```javascript
var file = request.files['file'] // `file` is the param name
var picture = backbeam.empty('file') // you can create a new object or you can use an existing one
picture.saveFile(file, function(err) {
    // file saved
})
```

Each object inside request.files has the following parameters: name, type and size

### The response object

Using the response object you can manipulate the HTTP response

If you want to send just a string you would call the response.send as follows

```javascript
response.send('Hello world')
```

If you want to render an view template you would use the response.render method. For exmaple:

```javascript
response.render('index.html', { events: some_objects })
```

The first argument is the name of the view you have created in the control panel. The second argument is optional and it should be a collection of key-value paris. These objects can be accessed from the referenced view to render the content of the response.

When you render a template by default a `Content-Type: text/html;charset=utf-8` header is set on the HTTP response

You can change the content-type response header using

```javascript
response.contentType('text/plain')
```

By default the response's HTTP status code is 200. You can change that using:

```javascript
response.status(418) // typical status codes are: 200, 201, 301, 302, 400, 403, 404, 500, 501,...
```

You can manipulate the response headers using get and set

```javascript
var currentValue = response.get('Cache-Control')
response.set('Cache-Control', 'no-cache, private, no-store, must-revalidate')
```

You can send a JSON response using repsonse.json. The `Content-Type` header is automatically set to `application/json;charset=utf-8` and the argument is automattically converted to JSON. Example:

```javascript
response.json({message: 'Hello world'})
```

If you need to redirect a user you can use the response.redirect method. It is as simple as follows:

```javascript
response.redirect('http://www.example.com')
```

You can also send files from your controllers. For example you can create a generic controller with the following path pattern: `/see/:id/:version` and then implement this code:

```javascript
backbeam.read('file', request.params.id, function(err, obj) {
  response.sendFile(obj, {width:100})
})
```

The controller needs the unique identifier of the file object and the version field value. The version param is not used in the code but you should include it in the path. The version value changes each time the file content changes, so including the version in the URL prevents the browser to cache one version of the file and not refreshing it if the file changes.

The `sendFile` method accepts two arguments: the first is the file object and the second one is an optional object containing options for the file. If the file is an image you can for example scale it setting its width. If you set the option values in code be aware that if you change them some users will see cached versions with old option values. It is better to change the controller path pattern when you change hard-coded transformation options.

### The session object

Backbeam provides a session objects based on cookies. You need to be aware that cookies have a limited size depending on the user's browser so you should only store minimal information. If you need to store more information concerning a user you should user the built-in backbeam database insted of the session object.

You can save information in the cookie just by calling the session.set method. Example

session.set('search-filters-enabled', false)
You can store any object that can be converted to JSON. To retrieve a stored value use session.get

```javascript
var filtersEnabled = session.get('search-filters-enabled')
```

You can delete a certain value using session.del(key) and you can delete all the values using session.destroy(). You tipically would use the session.destroy() method combined with a method call to backbeam.logout() if you are storing some information about the current logged user.

### Using libraries

Your server-side code will grow and you probably will need repetitive code in many controllers. In that case you can create libs that are just re-usable parts of code. The API to use libs is very similar to the node.js modules. This is done by design to keep the code as compatible as possible between node.js and bacbkeam. For example you can create a helpers.js lib with a code similar to this:

```javascript
exports.awesomeFunction = function(...) {...}
```

Then you use those utility functions froma controller or from another lib.

```javascript
var helpers = require('helpers')
helpers.awesomeFunction(...)
```

As in node.js packages you can also completely override the exports object as follows:

```javascript
module.exports = anything
```

### Sending objects to the SDKs

Instead of creating, querying or editing objects directly from the SDKs you can also write the business-logic in the server-side and send the objects to the SDK that you are using. This has some benefits:

The business logic is in the server-side so you can change it at any time without needing to update the mobile applications. And without the problem of having users using old versions of your application (and therefore running old business logic).
You can share business logic between platforms. You can write the business logic in your controllers and then call those controllers from iOS, Android, etc. without having to re-implement all the business rules in each platform.

You can write client applications without API keys using only methods that call your controllers. This way since you aren't using API keys you don't have the problem of anyone stoling your API keys.

It is very simple to send objects from a controller to a client application. Many server-side functions let you pass the response object instead of a callback. This way backbeam fills the response object with the result data and in a format that the client applications can understand. See the example:

```javascript
// pass the response object instead of a callback and the controller
// will send a JSON response that the SDKs will understand
backbeam.select('some-entity').fetch(100, 0, response)
```

Try the example above and execute it. You will see a JSON response with the common response format

This is a list of methods that support this functionality (passing the reponse object instead of a callback to generate a JSON response)

* object.save()
* object.refresh()
* object.remove()
* query.fetch()
* query.near()
* query.bounding()
* backbeam.login()
* backbeam.read()
* backbeam.verifyCode()
* backbeam.twitterSignup()
* backbeam.facebookSignup()

Additionally if you make some users authentication operation the controller will send special response headers to tell the SDK which user is now logged in or if the user has logged out. The SDK is also responsible of sending a special request header to tell the controller which is the logged user so from the controller you can just call `backbeam.currentUser()` as always to know the current logged user.

You can know from what SDK the controller has been invoked by using `backbeam.sdk()`. With this method you could also do conditional rendering. For example:

```javascript
var callback = null
if (backbeam.sdk()) {
  // generate an understable response if an SDK is invoking the controller
  callback = response
} else {
  callback = function(err, objects) {
    // render an HTML tempalte if the controller is invoked from the browser
    response.render('template.html', { objects:objects })
  }
}
backbeam.select('some-entity').fetch(100, 0, callback)
```

Or simply

```javascript
var callback = backbeam.sdk() ? response : function(err, objects) {
  response.render('template.html', { objects:objects })
}
backbeam.select('some-entity').fetch(100, 0, callback)
```

### Sending JSON to the SDKs

If you wan to have full control of the response format you can also send JSON:

```javascript
response.json({ hello: 'world' })
```

The good news is that all the side effects of the previous section (Sending objects to the SDKs) also applies when you send JSON responses. So all the users authentication stuff is included: the SDK tells the controller which user is authenticated and you can access it with `backbeam.currentUser()` and the controller will tell the SDK if the current logged user has changed or the user has logged out.

Learn how to invoke controllers from the SDKs: from Android and from iOS

### HTTP Client

In the server-side code you can make http requests if for some reason you need to integrate your backend with third-party services. This API supports HTTP and HTTPS and `gzip` and `deflate` compressed responses.

The way to create a new HTTP Client is as follows:

```javascript
var http = backbeam.httpClient()
And the way to make a request is the following:

http.request(options, function(err, res) {
    // handle the response
})
```

You can make multiple requests with the same HTTP Client. The only benefit of creating different HTTP Clients is that each one has a different cookie storage.

The options parameter is an object that can contain the following attributes:

* `url`
A string containing the URL to request. It is mandatory
method
The HTTP method. If not defined GET is used by default
* `qs`
An optional object with the query string parameters. For example { site: 'stackoverflow', sort: 'reputation', order: 'desc' }
* `headers`
An optional object to specify additional request HTTP headers
* `body`
An optional object that specifies the request body
* `form`
If present this object should be an object with key-value pairs to be encoded as application/x-www-form-urlencoded (the default encoding used in HTML when submitting POST forms)
* `followRedirect` (default true)
Boolean value that specifies if HTTP 3xx redirects should be followed. There is a maximum of 10 redirects
* `followAllRedirects` (default false)
Boolean value that specifies if no-HTTP redirects should be followed. Max 10 redirects if enabled.
* `auth`
An optional object for HTTP authentication that if defiend must contain a username and password. You can also pass a sendImmediately parameter (default to true). If set to true the authentication information is sent immediatly, otherwise a plain request is made waiting for a 401 response and then another request is made with the authentication information. Digest authentication is supported but you need to set sendImmediately to false.
* `encoding`
Optional parameter to specify the encoding to use to parse the response body. You can choose between utf-8, ascii or binary. If not present the HTTP client will inspect the Content-Type header to determine with very basic rules which encoding to use.

The response object contains just three attributes:

* `body`
The response body
* `headers`
An object with the response HTTP headers
* `statusCode`
The HTTP status code that returned the server

There are a few convenience methods that automatically specify the HTTP method to use:

* `httpClient.get()`
for GET requests
* `httpClient.post()`
for POST requests
* `httpClient.put()`
for PUT requests
* `httpClient.patch()`
for PATCH requests
* `httpClient.head()`
for HEAD requests
* `httpClient.del()`
for DELETE requests

The HTTP Client does not apply further response manipulation than decompressing gzip and deflate encodings. If you need to consume a JSON-based REST API for example you will probably need to parse the response body using `JSON.parse()`. It is not done by the HTTP Client since there are too many ways (some non-standard) to specify if a response contains JSON data. There are many MIME types that the third-party services could be using in the `Content-Type` header (`text/json`, `application/json`,...) and some services are starting to use their own MIME types such as `application/vnd.github.v3+json`. So just use `JSON.parse()` if you know that the service you are consuming is sending JSON or not. Let's see an example

```javascript
var http = backbeam.httpClient()
var url = 'https://api.stackexchange.com/2.1/users'
var qs = { site: 'stackoverflow', sort: 'reputation', order: 'desc' }
http.get({ url:url, qs:qs }, function(err, res) {
    var data = JSON.parse(res.body) // you could wrap this into a try-catch
    var items = data.items
    response.render('index.html', { items: items }, true)
})
```

### Sending emails

There are a few automatic email messages that are sent to the users in order to confirm their email addresses, or reset their password, for example. You can also create custom email templates in the control panel and you can send those email templates from the server-side code with the following method:

```javascript
backbeam.sendMail('template-identifier', { myOption: 'optionValue'}, 'usuario1@example.com', 'usuario2@example.com', 'usuario3@example.com')
backbeam.sendMail('template-identifier', { myOption: 'optionValue'}, userObject1, userObject2, userObject3)
backbeam.sendMail('template-identifier', { myOption: 'optionValue'}, userObjectsArray)
```

So you can use email addresses, user objects or an array of user objects.

The template-identifier is present in the URL when you edit the email template. For example if you are accessing the email template in the control panel in the following address:

```javascript
http://example.backbeam.io/conf/email/custom-example/txt
```

The part highlighted is the template identifier you need to use when using `sendMail`. And the second parameter is the options passed to the template, so in the template you can use.

```
This is the value passed to this template: {{ myOption }}
```

### Server-side utilities

In your web controllers the backbeam object comes with a few utility methods for common things you will need to do in some situations. All this functionality is grouped in a backbeam.util object.

#### Parsing and generating querystrings

Sometimes you will need to parse or generate a querystring. This is the format of the data after the ? character in a URL or the format of a HTTP message body whose `Content-Type` is `application/x-www-form-urlencoded`.

```javascript
var qs = backbeam.util.querystring.stringify({ foo: 123, bar: 456 }) // returns 'foo=123&bar=456'
var params = backbeam.util.querystring.parse('foo=123&bar=456') // returns { foo: '123', bar: '456' }
```

Note that the `parse()` method returns strings in this example. That's because this method won't do any type conversion.

#### Crypto

Sometimes you will need to communicate with external services (with the HTTP client for example) and sometimes you will need to calculate some signatures. With `backbeam.util.crypto` you can calculate hashes and hmacs.

You can call `backbeam.util.crypto.getHashes()` to obtain an array with the available algorithms. At this time you can use the following algorithms: `md5`, `sha1`, `sha256`, `sha384`, `sha512`.

To calculate a hash you will create an object using `backbeam.util.crypto.createHash(algorithm)`, then you will call `update()` one or more times and finally you will get the message digest using `digest(encoding)`.

```javascript
var hash = backbeam.util.crypto.createHash('sha1')
  .update('The quick brown fox jumps over the lazy dog')
  .digest('hex') // returns '2fd4e1c67a2d28fced849ee1bb76e7391b93eb12'
```

The digest encoding can be `hex`, `base64` or `binary`.

Calculating an HMAC is similar. You call `createHmac(algorithm, key)` and everything else is like calculating a hash.

```javascript
var hash = backbeam.util.crypto.createHmac('sha1', 'key')
  .update('The quick brown fox jumps over the lazy dog').digest('hex')
  .digest('hex') // returns 'de7c9b85b8b78aa6bc8a7a36f70a90701c9db4d9'
```
