# Manipulating data

## Queries

To create a query use `select()`. You must pass the entity identifier:

```javascript
var events_query = client.select('event')
```

Optionally you can use BQL queries. The first argument is the BQL query and the second parameter is optional. It can be a single value, an array of values or just multiple arguments

```javascript
events_query.query('join place') // BQL without parameters
events_query.query('where type=?', 'conference') // one parameter
events_query.query('where type=? and start_date>?', 'conference', new Date()) // multiple arguments
events_query.query('where type=? and start_date>?', ['conference', new Date()]) // array
```

Use `fetch()` to retrieve the results of the query. This method needs three arguments. The first is the limit of objects to be returned. The second is the offset, and the third argument is a callback function with three arguments. Callbacks always receive an error as the first parameter. This parameter is null if everything is ok. The second argument in this case is an array of objects (the result of the query). And finally you receive the total count of objects of the query (the total number of objects without pagination).

```javascript
events_query.fetch(10, 0, function(err, objects, totalCount, fromCache) {
  // do something with these objects
})
````

`objects` is the array of results and `totalCount` is the total number of objects matching the query. For example there could be 1000 objects matching the query but since you requested a maximum of 10 objects the `objects` array has 10 objects and `totalCount` is equal to 1000.

Query objects support chained method calls, so you can do everything in a single line:

```javascript
backbeam.select('event').query('join place').fetch(10, 0, function(err, objects, totalCount, fromCache) {
  // do something
})
```

## Collection constraints

There is a helper object to create collection constraints when using the in operator in a query.

```javascript
var collection = backbeam.collection(someUser)
          .addTwitter(someTwitterId)
          .addFacebook(someFacebookId)
          .addGooglePlus(someGooglePlusId)
          .addEmail('user@example.com')

backbeam.select('user').query('where this in ?', collection)
```

The `backbeam.collection()` method accepts one or many objects or even an array of objects. You can also use identifiers (strings) directly. If you want to add constraints related to users you can add Twitter, Facebook, Google+, LinkedIn or GitHub identifiers or email addresses using the methods: addTwitter, addFacebook, addGooglePlus, addLinkedIn, addGitHub and addEmail that also support one or many strings of identifiers or an array of identifiers.

## Querying and refreshing single objects

It is easy to refresh the data of a given object. You just need to call the refresh method.

```javascript
var obj = backbeam.empty('place', known_id)
obj.refresh(function(error) {
  // here the object is populated with its values if no error ocurred
})
```

## Creating and updating objects

You can create an empty object and then save it as follows:

```javascript
var obj = backbeam.empty('place')
obj.set('some-field', some_value)
obj.save(function(error) {
  // here the object has been created and you can access its id(), createdAt() and updatedAt() methods
})
```

## Deleting objects

You can delete an object using the `remove()` method.

```javascript
var obj = ... // some object from the database
obj.remove(function(error) {
  // object deleted if there were no errors
})
```

Do not confuse this mehtod with `rem(fieldName, object)`. The latter is used to remove an object from a relationship. We wanted to avoid the confusion using `delete` as name for the method to delete an object from the database, but `delete` is a keyword in JavaScript.

