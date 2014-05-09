# Manipulating data

This documentation is about manipulating data from your Android code. This requires you to create a pair of API keys and configure them in your main activity. If you want to user server-side logic you should go to the [Server-side logic](android-server-side-logic.md) section of the documentation.

## Making queries

You make queries using `Query` objects. You create them using the `Backbeam.select()` method passing the entity identifier. It is optional to set a BQL query using `setQuery()`. Then you fetch the results calling `fetch()` with three arguments:

* `limit` The maximum number of elements to fetch
* `offset` The offset of the query
* `callback` The callback handler to handle the query result

The `setQuery()` method receives a Stirng containing a BQL query. If the BQL query needs parameters you pass them to the method just separating them with commas.

```java
q.setQuery("where name like ? and score > ?", "awesome", 100);
```

All operations that need to send or receive information fromt the server are handled asynchronously. If the operation receives information you provide a callback. Since Java doesn't support closures yet, you normally will pass an object of an anonymous class created inline. These callbacks usually have a `success()` method that you must implement and optionally you can override a `failure()` method.

```java
Backbeam.select("place").setQuery("sort by created_at").fetch(100, 0, new FetchCallback() {
  @Override
  public void success(List<BackbeamObject> objects, int totalCount, boolean fromCache) {
    for (BackbeamObject object : objects) {
      // Do something with this object
      System.out.println("object "+object.getString("name"));
    }
  }
});
```

In this case the `success()` method receives three arguments

* objects The list of objects returned by the query
* totalCount The total number of objects that this query could return
* fromCache Indicates whether the result was fetched from the cache or not

Queries can be cached in different ways. Before fetching the result of a query you can set the query fetch policy calling the `setFetchPolicy()` method. There are a few fetch policies available:

* REMOTE_ONLY This is the default policy. The cache is ignored and the operation is performed against the server
* LOCAL_ONLY The result is fetched from cache if available. If not available the failure() method is called
* LOCAL_AND_REMOTE The result is fetched from cache if available. Then the query fetches the result from the server. So in this case the method success() might be called twice.
* LOCAL_OR_REMOTE The result is fetched from cache if available. If not available the result is fetched from the server.

No matter what policy you use either `success()` or `failure()` are guaranteed to be called at least once.

## Removing objects matching a query

If you need to remove several objects at once and they share common constraints you can create a query and then call one of these methods... You can remove a limited number of objects:

```java
Query query = Backbeam.select("some-entity").setQuery(...);
query.remove(10, 0, new RemoveCallback() {

  @Override
  public void success(int removed) {
    // Here the first 10 objects matching the query have been removed
  }

});
```

Or you can remove all objects matching the query

```java
Query query = Backbeam.select("some-entity").setQuery(...);
query.removeAll(new RemoveCallback() {

  @Override
  public void success(int removed) {
    // Here the objects have been removed. You can get the number of removed objects
  }

});
````

## Collection constraints

There is a helper object to create collection constraints when using the in operator in a query.

```java
CollectionConstraint collection = new CollectionConstraint();
collection.addTwitterIdentifier(someTwitterIdentifier);
collection.addObject(new BackbeamObject("user", someBackbeamIdentifier));
collection.addEmailAddress("user@example.com");

Query query = new Query("user");
query.setQuery("where this in ?", collection);
query.fetch(100, 0, new FetchCallback() {

  @Override
  public void success(List objects, int totalCount,
      boolean fromCache) {

    System.out.println("objects: "+objects.size());
  }
});
```

As you can see in the collection constraint you can use Backbeam objects. This applies to any entity in your data model. If you are querying users you can also use Twitter, Facebook, Google+, LinkedIn or Twitter identifiers as well as email addresses.

## Creating or updating objects

To create or update objects you use the save()method. To create a new object first you need to instantiate an empty object passing the entity name to which this object will be a member of.

```java
BackbeamObject obj = new BackbeamObject("place");
obj.setString("name", "A new place");
obj.save(new ObjectCallback() {
    @Override
    public void success(BackbeamObject object) {
        System.out.println("created! :) "+object.getId());
    }
});
```

* `setString(String fieldName, String value)` Sets the value of a text field
* `setDate(String fieldName, Date value)` Sets the value of a date field
* `setNumber(String fieldName, Number value)` Sets the value of a numeric field
* `incrementNumber(String fieldName, int value)` Increments the value of a numeric field
* `setLocation(String fieldName, Location value)` Sets the value of a "location" field
* `setBoolean(String fieldName, boolean value)` Sets the value of a boolean field
* `setDay(String fieldName, GregorianCalendar value)` Sets the value of a "day" field. You can pass a Gregorian calendar. You can also pass a java.util.Date and the current calendar's timezone will be used to calculate the year, month and day.
* `setObject(String fieldName, BackbeamObject value)` Sets the value of a relationship "to-one"

If you want to update an object of which you know the identifier but you don't have an instance of it you can pass the identifier to the `BackbeamObject` constructor

When an object is created or updated the server saves the changes you made and sends you any other change that could been made to that object. So inside the `success()` method the object is fully synchronized.

```java
BackbeamObject obj = new BackbeamObject("place", identifierYourAlreadyKnow);
obj.setString("name", "A new name for that palce");
obj.save(new ObjectCallback() {
    @Override
    public void success(BackbeamObject object) {
        // here you can access any field of 'object'
    }
});
```

To add or remove objects of a relationship "to-many" you will use the methods `removeObject(String fieldName, BackbeamObject object)` and `addObject(String fieldName, BackbeamObject object)`. For example:

```java
BackbeamObject user = ...;
user.addObject("friends", myNewFriend);
user.removeObject("friends", userThatIsNoLongerMyFriend);
user.save(new ObjectCallback() {
    @Override
    public void success(BackbeamObject object) {
        // ...
    }
});
```

## Reading and refreshing objects

If you have an object instance and you need to fill it with the latest changes on the server without making you new changes then you use the refresh() method.

```java
BackbeamObject obj = someObjectYouAlreadyHaveInstantiated;
obj.refresh(new ObjectCallback() {
    @Override
    public void success(BackbeamObject object) {
        // here you can access any field of 'object'
    }
});
```

The `refresh()` method is overloaded and accepts an optional first parameter that is part of a BQL query. In this parameter you can specify to join some relationships while synchronizating the state of this object.

```java
BackbeamObject obj = someObjectYouAlreadyHaveInstantiated;
obj.refresh("join some-relationship", new ObjectCallback() {
    @Override
    public void success(BackbeamObject object) {
        // here you can access any field. You can also access the joined relationship
        JoinResult result = object.getJoinResult("some-relationship")
    }
});
```

If you need to read an object of which you know its identifier and you don't already instantiated it then you can use `Backbeam.read()`.

```java
Backbeam.read("place", id, new ObjectCallback() {
    @Override
    public void success(BackbeamObject object) {
        // ...
    }
});
```

The first argument is the entity identifier, the second is the unique identifier of that object and finally you pass the callback that will be called when the operation is complete. You can optionally pass a parameter that is part of a BQL query to join some relationships.

## Working with files

There is a special kind of entity that lets you hold files. It is the File entity. The entity itself holds information about the type of file, size, etc. If you need the download the content of the file you need to generate a URL. There is a special method called `composeFileURL` to which you can pass parameters. For example if you want to download an image you can transform it indicating a width and/or height. You can also pass `null` to this method.

```java
TreeMap<String, Object> options = new TreeMap<String, Object>();
options.put("width", 40);
options.put("height", 40);
String url = fileObject.composeFileURL(options);
````

Note that the file object should have their properties loaded. Specially `File` objects have a `version` field that is required to generate the URL. This field changes everytime the file content changes. This way the URL changes everytime the content changes and there are no problems with any cache. In summary, be sure that the file has its version field loaded. You can't do a refresh if it is not loaded. And if the file object is the result of a query be sure to use a join on the relationship. See this example:

```java
Backbeam.select("company").setQuery("join logo").fetch(100, 0, new FetchCallback() {
  @Override
  public void success(List<BackbeamObject> companies, int totalCount, boolean fromCache) {
    for (BackbeamObject company : companies) {
      BackbeamObject logo = company.getObject("logo");
      String logoURL = logo.composeFileURL(null);
      // Do something with logoURL
    }
  }
});
```

For images it is recommended to use an external library such as [Android Universal Image Loader](https://github.com/nostra13/Android-Universal-Image-Loader) to handle images efficiently.

## Geoqueries

You can perform geoqueries on entities having at least one field whose type is set to location. There are two types of geolocated queries: you can query the closest objects to a given point or you can query the objects inside a bounding box.

### Fetching the closest objects to a location

To query objects near to a given location you need to tell the SDK the name of the field whose type is location, the coordinates of the point (latitude + longitude), and the maximum number of results you want to fetch. The success block is very similar to a regular success query block but it returns additionally the distances of the objects to the location used to query the data. The distances are measured in meters.

```java
Query query = new Query("place");
query.near("location", 41.641113, -0.895115, 10, new NearFetchCallback() {

  @Override
  public void success(List<BackbeamObject> objects, int totalCount,
      List<Integer> distances, boolean fromCache) {

    // Success

  }
});
```

### Fetching objects in a bounding box

To query objects inside a bounding box you need to tell the SDK the name of the field whose type is location, the south-west and north-east coordinates of the corners of the bounding box, and the maximum number of results you want to fetch. The success block is exactly the same to any regular success query block.

```java
Query query = new Query("place");
query.bounding("location", 41.641113, -0.902896, 41.645883, -0.895115, 10, new FetchCallback() {

  @Override
  public void success(List<BackbeamObject> objects, int totalCount,
      boolean fromCache) {

    // Success

  }
});
```
