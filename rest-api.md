# REST API


## General considerations

The REST API is designed to be as simple as possible but no more.

* It always returns JSON (or JSONP). There are no plans to support other formats.
* It respects most of the HTTP semantics but it is not an Hypermedia API.
* It uses a custom authorization system similar to OAuth or some services from Amazon Web Services.
* It supports JSONP so you can do API calls from the browser.

## Content-Types, encoding and HTTP status

The API always returns `Content-Type: application/json; charset=utf-8` (or `text/javascript` for JSONP). It always returns UTF-8 and it always expects information to be received as UTF-8.

The API can return several HTTP status codes.

* It will respond with 20x if everything goes ok
* If you are indicating a project, entity or object that does not exist it will respond with a status code 404
* It will respond with other 40x status codes if you send bad requests (bad API key, bad parameters, etc.)
* It will return 50x if an internal error occurs.

## URLs

Backbeam provides two subdomains to use the REST API of your project. The pattern is the following:

`http://api-<env>-<your-project>.backbeamapps.com`

The `env` is either `dev` or `pro` and the following part of the pattern is the identifier of your project. This identifier is the identifier you already see as subdomain when you are in the control panel.

## API authorization

The authorization system is inspired on some mechanisms from OAuth and APIs like Amazon Web Services but it is a custom authorization system. It requires a pair of keys that you can create in your project. Notice that development and production keys are different.

Each API request has at least 4 parameters:

* `nonce` A unique identifier that needs to be between 30 and 42 characters long
* `time` A timestamp used to prevent repetible requests. Be aware that if your system clock is out of sync by many seconds the request will be rejected.
* `key` The shared key of your pair of keys
* `signature` A signature calculated using the parameters of the request and the private key

## Signature calculation

The signature parameter is calculated using the parameters of the request. The parameters must be sent in the querystring of the URL for GET requests. For any other request parameters must be sent in the request body.

To calculate the signature first generate a string concatenating the parameters in the following manner:

`param1=value 1&param2=value 2&param3=value 3&param4=multi value 1&param4=multi value 2`

This string looks like a querystring of an URL but it is not. It does not have its values escaped. Additionally when making API calls this string must include always at least two parameters relative to the request: a method parameter with the HTTP method and a path parameter with the path of the URL you are requesting.

Parameters must be sorted in alphabetical order. If a parameter has multiple values append them as in the example above (sorting the values with alphabetical order). Once generated this string you just need to calculate an HMAC-SHA1 hash using the calculated string and the secret key of the key pair.

The signature calculation has one exception: when requesting a file to download you don't include the nonce nor the time parameters. This is to prevent the URL to change in time. So it can be cached properly by the HTTP caching mechanisms.

## JSONP support

The Backbeam REST API supports JSONP. You can pass a `_method` and a callback parameter to any request. If you add a `_method` parameter that value would be taken as the HTTP method to perform the request. If you specify a callback parameter then the JSON response will be wrapped in a function call and the content-type of the request will be set to `text/javascript; charset=utf-8`.

These parameters are ignored in the signature calculation. If you or the HTTP lib/framework you are using sends a "_" parameter it is also ignored. This parameter is common in some frameworks (like jQuery) to include a timestamp and thus generate always a unique URL and prevent any caching side-effect.

When you indicate a callback parameter (and thus you are asking for JSONP support) the API always returns status code 200 since you won't be able to handle the response properly in other case.

## Response structure

Backbeam always returns a JSON object (`{}`) with a status parameter. If everything is ok status will be Success. In any other case this attribute indicates the reason of the error.

In most cases the API returns objects and all the responses have the same structure. There is an objects attribute even if it only returns the information of one object. This attribute is an object whose keys are the identifiers of the objects. The values are objects indicating the fields and values of those objects.

If the operation semantically only returns an object then there will be an id attribute with the unique identifier of that object. If the operation semantically could return more than object then there will be an ids attribute with an array of ids. When the operation has a query then it is also returned a counter named count indicating how many objects match the given query regardless of the number of returned objects (because you can limit the number of objects returned).

```javascript
{
  "status": "Success",
  "objects": {
    "uDiuuGIMMQ4O": {
      "name#t": "Name of this place",
      "description#rt": "The <em>description</em> of this place",
      "location#l": {
        "lat": 41.64364630414023,
        "lon": -0.8893990516662598,
        "hash": "ezrkgub",
        "addr": "Street address of this location"
      },
      "type#s": "Pub",
      "created_at": 1368520407344,
      "updated_at": 1368520407344,
      "type": "place"
    }
  },
  "ids": [
    "uDiuuGIMMQ4O"
  ],
  "count": 57
}
```

When you perform a query with joins then the objects attribute contains the main list of objects as well as the objects returned by the joins.

There are some fields that are present in all objects:

* `type` indicates to which entity this object is member of.
* `created_at` contains a timestamp which is the date when this object was created.
* `updated_at` contains a timestamp which is the date when this object was updated for the last time (or created if it has never been updated).

The rest of the fields are defined by you in the control panel. In the JSON response they appear with their names followed by `#` and one or more characters. These characters indicate the data type of that field. JSON supports some data types such as strings or numbers but lacks of other data types such as dates. A date for example can be expressed as a string or as a number in a JSON object but this introduces a problem: if you don't know the semantics of a field in advance you don't know if a field is representing a date, a number or a string that may look like a date. Having these information with the field name removes this ambiguity. These are the available data types and the characters that indicate their presence:

* A text field is indicated with `#t`.
* A textarea field is indicated with `#ta`.
* A richtextarea field is indicated with `#rt`.
* A date field is indicated with `#d` and it is represented with a numerical timestamp.
* A number field is indicated with `#n`.
* A select field is indicated with `#s`  and it is represented with an string.
* A location field is indicated with `#l` and it is represented with an object with some fields like `lat`, `lon`, `hash` and `addr`.
* A relationship field is indicated with `#r` and it is represented in several ways.
* A select field is indicated with `#s` and it is represented with an string.

If a field is a relationship "to-one" and you haven't performed a join over it then the API returns for that field an object with two attributes: `id` with unique identifier of that object and `type` indicating the identifier of the entity this object is member of.

```javascript
"author#r": {
    "id": "uDiuuGIMMQ4O",
    "type": "user"
}
```

This is very useful if you are writing an SDK because you don't receive the information of the object (because you haven't joined it) but you know the identifier an the entity of the object, so you can create an empty object ready to be used for future operations.

If you perform a join over that field then the API will return just one string indicating the unique identifier of the object. This object will be present in the objects attribute.

`"author#r": "uDiuuGIMMQ4O"`

If a field is a relationship "to-many" and you have performed a join over that field then the API returns for that field an object with two attributes: result with an array of identifiers and count indicating the total of objects in the relationship.

```javascript
"images#r": {
    "result": [
        "mDWuly5NaQxO"
    ],
    "count": 5
}
```

The joined objects will be present in the objects attribute.

## Commands

When you create or update objects you send commands. Each command performs an action over a field value. For example when you want to set the value of a field "name" you send a "set" command set-field. These are the available commands:

* `set-<fieldid>` Sets the value of that field. Not applicable to relationships
* `incr-<fieldid>` Increments the current value of this field. Only applicable to numbers
* `add-<fieldid>` Adds an object to a collection. Only applicable to relationships "to-many"
* `rem-<fieldid>` Removes an object from a collection. Only applicable to relationships "to-many"
* `del-<fieldid>` Deletes the value of that field. Not applicable to relationships. In this case the value is ignored, but should not be an empty string. Send just "-" as value.

## API endpoints for data manipulation

There are several API endpoints to query and manipulate data.

### List objects of a given entity

`GET /data/:entity`

**Parameters**

* `q` (optional): BQL query to determine which objects return
* `params` (optional): parameters required by the BQL query
* `limit` (optional): maximum number of objects to return
* `offset` (optional): indicates the offset of the first object to return

**Description**

This URL returns a list of objects of a given entity optionally filtered by a BQL query.

### List objects of a given entity near to a given geographical location

`GET /data/:entity/near`

**Parameters**

* `q` (optional): BQL query to determine which objects return
* `params` (optional): parameters required by the BQL query
* `near` (required): field identifier to be used to calculate the nearest objects from
* `lat` (required): latitude coordinate
* `lon` (required): longitude coordinate
* `limit` (optional): maximum number of objects to return

**Description**

This URL returns a list of the nearest objects from a given location optionally filtered by a BQL query. This

### List objects of a given entity inside a geographical bounding box

`GET /data/:entity/bounding`

**Parameters**

* `q` (optional): BQL query to determine which objects return
* `params` (optional): parameters required by the BQL query
* `near` (required): field identifier to be used to calculate the nearest objects from
* `swlat` (required): south-west latitude coordinate
* `swlon` (required): south-west longitude coordinate
* `nelat` (required): north-east latitude coordinate
* `nelon` (required): north-east longitude coordinate
* `limit` (optional): maximum number of objects to return

**Description**

This URL returns a list of objects contained in a geographical bounding box optinally filtered by q BQL query. The bounding box is defined by two coordinates: the south-west coordinate and the north-east coordinate.

### Read an existing object

`GET /data/:entity/:id`

**Parameters**

* `joins (optional)`: part of a BQL query to perform joins over this object
* `params (optional)`: parameters required by the BQL query

**Description**

This URL returns an object given its entity and unique identifier. Optionally you can porform some joins to fetch additional information.

### Create an object

`POST /data/:entity`

**Parameters**

All parameters in this request are treated as 'commands'. See the commands section.

**Description**
This URL creates a new object of the given entity and returns the data of that object like in a GET operation.

### Update an object

`PUT /data/:entity/:id`

**Parameters**

All parameters in this request are treated as 'commands'. See the commands section.

**Description**

This URL updates an existing object and returns the data of that object like in a GET operation.

### Delete an object

`DELETE /data/:entity/:id`

**Parameters**

No parameters

**Description**

This URL deletes an existing object and returns the data of that object like in a GET operation.

## API endpoints for users authentication

There are a few API endpoints to authenticate users.

### Authenticate a user using email+password

`POST /user/email/login`

**Parameters**

* `email` (required): the email of the user
* `password` (required): the password of the user
* `q` (optional): part of a BQL query to perform joins over the user if the login succeds
* `params` (optional): parameters needed by the query

**Description**

This URL authenticates the user using email+password. If the login succeeds an auth attribute is returned.
