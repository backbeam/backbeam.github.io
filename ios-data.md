# Manipulating data

This documentation is about manipulating data from the iOS code. This means you are using client-side logic so it requires you to create a pair of API keys and configure them in your `AppDelegate`. If you want to use server-side logic you should go to the [Server-side logic](ios-server-side-logic.md) section of the documentation.

## Queries

The most common thing you will want to do is making queries to the database from your code. This requires you to use `BBQuery` objects. You need to specify which entity you want to query and then most of the times you will specify the BQL query and parameters, optionally a fetch policy and finally you will fetch the data. Let's see an example:

```objectivec
BBQuery* query = [Backbeam queryForEntity:@"entity-name"];
// optional: use a BQL query for constraints
[query setQuery:@"where ..." withParams:[NSArray ...]];
// optional: you can set a fetch policy
[query setFetchPolicy:BBFetchPolicyLocalAndRemote];
[query fetch:100 offset:0 success:^(NSArray* objects, NSInteger totalCount, BOOL fromCache) {
  // do something with these objects
} failure:^(NSError* error) {
  // something went wrong
}];
```

The fetch policy indicates if the local cache should be used and how. These are the the available values:

* `BBFetchPolicyRemoteOnly`
This is the default. Data is only retrieved from the server
* `BBFetchPolicyLocalOnly`
Data is only retrieved from the local cache. If no data is available the failure block is called
* `BBFetchPolicyLocalAndRemote`
Data is retrieved first from the local cache. If there is data available the success block is called. Then data is fetched from the server, so the success can be called twice.
* `BBFetchPolicyLocalOrRemote`
If data is available in the local cache it is passed to the success block. If it isn't data is fetched from the server. No block is called twice in either case.
* `BBFetchRemoteAndStore`
Fetch the data from the server and then update the cache with that information. This is useful if you know that the cache information is not up to date and you want to fetch the latest information and then update the cache. For example after deleting or updating an object in a list and you want to refresh that list.

## Collection constraints

There is a helper object to create collection constraints when using the `in` operator in a query.

```objectivec
BBCollectionConstraint *collection = [[BBCollectionConstraint alloc] init];
[collection addTwitterIdentifier:someTwitterIdentifier];
[collection addObject:[Backbeam emptyObjectForEntity:@"user" withIdentifier:someBackbeamIdentifier]];
[collection addEmailAddress:@"user@example.com"];

BBQuery *query = [Backbeam queryForEntity:@"user"];
[query setQuery:@"where this in ?" withParams:@[collection]];
[query fetch:100 offset:0 success:^(NSArray *objects, NSInteger total, BOOL fromCache) {
    NSLog(@"objects %d", objects.count);
} failure:^(NSError *error) {
    NSLog(@"error %@", error);
}];
```

As you can see in the collection constraint you can use Backbeam objects. This applies to any entity in your data model. If you are querying users you can also use Twitter, Facebook, Google+, LinkedIn or GitHub identifiers as well as email addresses.

## Removing objects matching a query

If you need to remove several objects at once and they share common constraints you can create a query and then call one of these two avaiable methods.

You can remove a limited number of objects:

```objectivec
BBQuery *query = [Backbeam queryForEntity:@"some-entity"];
[query setQuery:@"where something > ?" withParams:@[ @100 ]]; // some constraints
[query removeObjects:10 offset:0 success:^(NSInteger removed) { // remove the first 10 objects that match the query
    // Here the objects has been removed and you can access the number of removed objects
} failure:^(NSError *error) {
    // Something went wrong
}];
```

Or you can remove all objects matching the query

```objectivec
BBQuery *query = [Backbeam queryForEntity:@"some-entity"];
[query setQuery:@"where something > ?" withParams:@[ @100 ]]; // some constraints
[query removeAllObjects:^(NSInteger removed) { // remove all the objects matching this query
    // Here the objects has been removed and you can access the number of removed objects
} failure:^(NSError *error) {
    // Something went wrong
}];
```

## Querying and refreshing single objects

You can create empty objects of a certain entity by using these methods.

```objectivec
+ (BBObject*)emptyObjectForEntity:(NSString*)entity;
+ (BBObject*)emptyObjectForEntity:(NSString*)entity
           withIdentifier:(NSString*)identifier; // use this if you know the object id
```

There are a couple of reasons to create empty objects. The first one is obvious: you want to create a new one. Once instantiated you will be able to save it as you will see below. The other reason is when you know the identifier of an object and you want to add it to a relationship for example, but you don't need the actual data: you just want to create the relationship between both objects. In this case it is enough to pass an empty object as long as it has the right entity and identifier.

If you have an object that comes from somewhere and at some point you want to refresh its values because it could be out of date and you need the most recent information you can use the `refresh` method. There are two versions of this method. The first one only requires a succcess and a failure block. In this case the object have all its regular fields refreshed. The second version also allows you to perform joins over the object. Example:

```objectivec
[object refresh:@"join something having foo > ?" params:@[param] success:^(BBObject *object) {
  // do something on success
} failure:^(BBObject *object, NSError *err) {
  // ops, something went wrong
}];
```

You will only need the params argument if your BQL join needs params. Otherwise you can pass nil safely.

This second version of the refresh method will refresh all the regular fields and will refresh the relationships used in the BQL query as well.

## Creating and updating objects

To create or update objects just use the `save` method. Let's see how to create an object from scratch:

```objectivec
BBObject* object = [Backbeam emptyObjectForEntity:@"some_entity"];
[object setString:@"field value" forField:@"field-name"];
[object save:^(BBObject* obj) {
  // The object is saved! You can access its identifier if you need it
  // 'obj' and 'object' are the same object.
  // It is passed to the block just to simplify the code and prevent retain cycles
} failure:^(BBObject* obj, NSError* error) {
  // Something went wrong
}];
```

If you are updating an object and everything goes ok all fields are synchronized with the database, not just the fields you have changed. Let's see how it works:

```objectivec
BBObject* object = ...; // some object already created
[object setString:@"field value" forField:@"field-name"];
[object save:^(BBObject* obj) {
  // If someone changed this object the 'object' variable is now up to date with all those changes
  // For example, if someone changed a field called 'whatever' you can access its value here
  NSString* value = [obj stringForField:@"whatever"];
} failure:^(BBObject* obj, NSError* error) {
  // Something went wrong
}];
```

You have many methods to set the values of the fields depending on its value type:

```objectivec
// String fields
- (BOOL)setString:(NSString*)obj forField:(NSString*)key;

// Number fields
- (BOOL)setNumber:(NSNumber*)obj forField:(NSString*)key;

- (void)incrementField:(NSString*)key by:(NSInteger)value;

// Location fields
- (BOOL)setLocation:(BBLocation*)obj forField:(NSString*)key;

// relationship "to-one"
- (BOOL)setObject:(BBObject*)obj forField:(NSString*)key;

// Date fields
- (BOOL)setDate:(NSDate*)obj forField:(NSString*)key;

// Boolean fields
- (BOOL)setBoolean:(NSNumber*)obj forField:(NSString*)key;

// Day fields
- (BOOL)setDay:(NSDateComponents*)obj forField:(NSString*)key;

// Helper method for day fields (uses the current calendar timezone)
- (BOOL)setDayFromDate:(NSDate*)date forField:(NSString*)key;

// JSON fields
- (BOOL)setJSON:(id)obj forField:(NSString*)key;

// For any field type
- (BOOL)setRawValue:(id)obj forField:(NSString*)key;
```

We have seen how to create and update objects using simple fields, but what about relationships? You can manage "to-many" relationships using the following methods:

```objectivec
BBObject* user = ...; // some user
[user addObject:otherUser forField:@"contacts"]; // add 'otherUser' to 'contacts' relationship
[user removeObject:yetAnotherUser forField:@"contacts"]; // remove 'yetAnotherUser' from 'contacts' relationship
[user save:^(BBObject* obj) {
  // done!
} failure:^(BBObject* obj, NSError* error) {
  // Something went wrong
}];
```
