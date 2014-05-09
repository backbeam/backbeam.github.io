# Objects and query results

Regardless you are using client-side or server-side code you will handle information wrapped in objects. In this section you will learn the basic classes and methods available to access that information.

> If you are going to write server-side logic and you
> will always return JSON to the client-side code then
> You are not interested in these topics about objects
> and query results. But you are interested in the
> documentation about Json objects in the end of this
> section.
> After that, check the [server-side logic section](android-servers-side.md) to learn how to
> invoke the server-side logic from your Java code

## Objects

Most of the time you will read and manipulate the data stored in the database using `BackbeamObject`. This is a very basic class that encapsulates the information of a record in your database. You can access its fields and some other metainformation. These are the most important methods in the `BackbeamObject` class:

* `String getString(String field)` Returns the value of the given field if it is a String. If it is not present or it is not a String this method returns null
* `String getId()` Returns the unique identifier of this object or null if this object hasn't been stored in the database yet
* `Date getCreatedAt()` Returns the date in which this object was created or null if this object hasn't been stored in the database yet
* `Date getUpdatedAt()` Returns the date in which this object was created or null if this object hasn't been stored in the database yet
* `Date getDate(String fieldName)` Returns the value of a date field
* `Number getNumber(String fieldName)` Returns the value of a numeric field
* `Json getJson(String fieldName)` Returns the value of a JSON field
* `Boolean getBoolean(String fieldName)` Returns the value of a boolean field
* `GregorianCalendar getDay(String fieldName)` Returns the value of a "day" field
* `BackbeamObject getObject(String fieldName)` Returns the value of a relationship "to-one"
* `Location getLocation(String fieldName)` Returns the value of a "location" field
* `String getEntity()` Returns the identifier of the entity this object is member of
* `boolean isNew()` Returns true if this object hasn't been stored yet
* `boolean isEmpty()` Returns true if this object does not have any field value defined yet
* `boolean isDirty()` Returns true if there are pending changes in this object to be saved

You have equivalent methods to set the values of each field depending on its value type, but you only need them when using client-side business logic. You will learn more about that in the section about [manipulating data](android-data.md).

## Join results

Many times you will perform joins in your BQL queries and other operations. If the join is made in a relationship "to one" you can just use obj.getObject(fieldname) to get the value of the relationship. Let's see an example where the BQL query is made in the client side:

```java
Backbeam.select("event")
    .setQuery("join place")
    .fetch(100, 0, new FetchCallback() {

      @Override
      public void success(List<BackbeamObject> objects,
        int totalCount, boolean fromCache) {
        // pick a place (in real code check the objects.size() first)
        BackbeamObject event = objects.get(0);
        BackbeamObject place = event.getObject("place");
      }
});
```

You can also make joins in the "to many" side of a relationship. In that case you can fetch a number of objects in the relationship and you can always get the total number of objects in the relationship.

```java
Backbeam.select("place")
    .setQuery("join last 10 events")
    .fetch(100, 0, new FetchCallback() {

      @Override
      public void success(List<BackbeamObject> objects,
        int totalCount, boolean fromCache) {
        // pick a place (in real code check the objects.size() first)
        BackbeamObject place = objects.get(0);
        JoinResult join = place.getJoinResult("events");

        List<BackbeamObject> events = join.getResults();
        int count = join.getCount();
      }
});
```

If you are only interested in the number of objects in a relationship you can just query using for example join events. Check the [BQL documentation](overview-query-language.md) for further information.

## Users authentication

One of the functionalities of the SDK is to remember the authenticated user between sessions of the application. You can always access the current authenticated user using `Backbeam.currentUser()` and you can logout the user by calling `Backbeam.logout()`. This applies to all authentication mechanisms.

## Json objects

Backbeam comes with a JSON implementation. This implementation is contained in a sigle class: `io.backbeam.Json`. A `Json` object wraps any kind of JSON structure. This is, a Json object can contain a boolean value, a `String`, a `Map`, a `List`, a number or `null`. You can check wich type of object is wrapped using its `isXXX()` methods: `isString()`, `isNumber()`, `isBoolean()`, `isMap()`, `isList()` or `isNull()`.

You are interested in these kind of objects in two circumstances. If you are using the JSON field type the SDK will serialize and deserialize using the `Json` class. And if you are invoking server-side logic that returns data in the JSON format you will receive the information encapsulated in this type of object.

### Manually serializing and deserializing

Usually you will only want to create and use `Json` objects and you won't care about the serialization and deserialization. But if you are interested it's pretty simple.

This way you can parse a `String` that contains JSON:

```java
Json data = Json.loads("[1, 2, 3, null, true]");
```

And just calling the `toString()` method generates a `String` in JSON format with the content of the `Json` object.


### Objects / Maps

You can create a JSON object / map as follows:

```java
Json json = Json.map()
        .put("key1", 1234)
        .put("key2", true)
        .put("key3", null)
        .put("key4", "foo");
System.out.println(json);
````

The output:

```javascript
{"key1": 1234, "key2": true, "key3": null, "key4": "foo"}
```

The `put(String key, Object value)` method accepts `Json` objects, primitive types, `Strings` and collections.

To access the information of a JSON object / map you can use the `Json get(String key)` method:

```java
int value1 = json.get("key1").asInt();
```

You have several `asXXX()` methods that return standart types (`boolean`, `int`, `String`, etc).

### JSON arrays / lists

You can create a Json array / list as follows:

```java
Json json = Json.list("foo", [], {}, true, false, null, 1234);
```

The output:

```javascript
["foo", [], {}, true, false, null, 1234]
```

You can access the elements of the list using the `Json at(int index)` method.

```java
int size = json.size();
boolean b = json.at(3).bool();
```

You can add elements to the underlying list using `add(Object...objects)`. You can pass `Json` objects, primitive types, `Strings` or collections.

### Complex example:

You can use the `import static` statement and use `list()` and `map()` directly. Which lets you do complex JSON structures with less code:

```java
import static siena.Json.*;
...

Json json = list("foo", "bar", "baz",
        map().put("key1", true)
                .put("key2", false)
                .put("key3", list(1, 2, 3, 4)));
System.out.println(json);
```

The output:

```javascript
["foo", "bar", "baz", {"key1": true, "key2": false, "key3": [1, 2, 3, 4]}]
```

### Converting from arrays and collections

You can also create `Json` objects from plain Java arrays or collections. The Json constructor accepts arrays, collections and maps. Examples:

```java
Json fromArray = new Json(new Object[]{1, 2, 3});
Json fromCollection = new Json(Arrays.asList(new Object[]{1, 2, 3}));
Map<String, Object> m = new HashMap<String, Object>();
m.put("foo", 1);
Json fromMap = new Json(m);
````

## Where to go next

If you are using client-side business logic then you should learn: [how to mamipulate data](android-data.md), [users authentication](android-authentication.md), and [push notifications](android-push-notifications.md).

If you think that server-side logic is better for your needs then you should learn [how to invoke the server-side logic from your Java code](android-server-side-logic.md).
