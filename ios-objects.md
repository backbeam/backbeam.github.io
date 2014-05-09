# Objects and query results

Regardless you are using client-side or server-side code you will handle information wrapped in objects. In this section you will learn the basic classes and methods available to access that information.

> The only case in which you are not interested in this
> section is when you write server-side logic and you
> always return JSON to the client-side code. In that
> case you don't receive backbeam objects in your code.
> You will receive the JSON data structure returned by
> the server that will be composed of NSDictionaries,
> NSArrays, NSStrings, etc.
>
> Check the [server-side logic section if this is your case](ios-servers-side.md)

## Objects

Most of the time you will read and manipulate the data stored in the database using `BBObjects`. This is a very basic class that encapsulates the information of a record in your database. You can access its fields and some other metainformation. These are the most important methods in the `BBObject` class

```objectivec
- (NSString*)identifier; // unique identifier of this object
- (NSString*)entity; // entity name
- (NSDate*)createdAt; // NSDate object indicating the created time
- (NSDate*)updatedAt; // NSDate object indicating the last updated time

// Methods to get values of fields of a certain type

// For text, textarea and rich-text fields
- (NSString*)stringForField:(NSString*)field;

// For date fields
- (NSDate*)dateForField:(NSString*)field;

// For number fields
- (NSNumber*)numberForField:(NSString*)field;

// For relationships "to-one"
- (BBObject*)objectForField:(NSString*)key;

// For location fields
- (BBLocation*)locationForField:(NSString*)key;

// For joined results in a query
- (BBJoinResult*)joinResultForField:(NSString*)key;

// For boolean fields (use NSNumber.booleanValue)
- (NSNumber*)booleanForField:(NSString*)key;

// For day fields
- (NSDateComponents*)dayForField:(NSString*)key;

// For JSON fields
- (id)JSONForField:(NSString*)key;

// If you want the value of a field no matter what type it is
- (id)rawValueForField:(NSString*)key;
```

You have equivalent methods to set the values of each field depending on its value type, but you only need them when using client-side business logic. You will learn more about that in the section about manipulating data

`BBObjects` also has some useful methods to know the state of the objects.

```objectivec
- (BOOL)isEmpty; // returns YES if this object has zero field values

- (BOOL)idDirty; // returns YES if the object was changed and not yet saved

- (BOOL)isNew; // returns YES if the object hasn't been saved for the first time
```

## Join results

Many times you will perform joins in your BQL queries and other operations. If the join is made in a relationship "to one" you can just use `[object objectForField:@"fieldName"]` to get the value of the relationship. Let's see an example where the BQL query is made in the client side:

```objectivec
BBQuery* query = [Backbeam queryForEntity:@"event"];
[query setQuery:@"join place"];
[query fetch:100 offset:0 success:^(NSArray* objects, NSInteger totalCount, BOOL fromCache) {
    BBObject *event = objects[0]; // pick a place (in real code check the array.count attribute first)
    BBObject *place = [event objectForField:@"place"];
} failure:^(NSError* error) {
    // something went wrong
}];
```

You can also make joins in the "to many" side of a relationship. In that case you can fetch a number of objects in the relationship and you can always get the total number of objects in the relationship.

```objectivec
BBQuery* query = [Backbeam queryForEntity:@"place"];
[query setQuery:@"join last 10 events"];
[query fetch:100 offset:0 success:^(NSArray* objects, NSInteger totalCount, BOOL fromCache) {
    BBObject *place = objects[0]; // pick a place (in real code check the array.count attribute first)
    BBJoinResult *join = [event joinResultForField:@"events"];

    NSArray *events = join.objects; // this is an array of BBObjects with the joined events
    NSInteger count = join.count; // this is the total count of objects in the relationship
} failure:^(NSError* error) {
    // something went wrong
}];
```

If you are only interested in the number of objects in a relationship you can just query using for example join events. Check the (BQL documentation)[overview-query-language.md] for further information.

## Users authentication

One of the functionalities of the SDK is to remember the authenticated user between sessions of the application. You can always access the current authenticated user using `[Backbeam currentUser]` and you can logout the user by calling `[Backbeam logout]`. In further sections you will see how to authenticate your user, but these methods apply to all of the authentication mechanisms.

If the user has been authenticated using an external provider (Twitter, Facebook, etc.) you can access information of that user from the external identity providers by using some of these methods:

```objectivec
- (NSString*)facebookData:(NSString*)key;
- (NSString*)twitterData:(NSString*)key;
- (NSString*)googlePlusData:(NSString*)key;
- (NSString*)linkedInData:(NSString*)key;
- (NSString*)gitHubData:(NSString*)key;
```

For example you can access the id, name or link of a user authenticated with Facebook. If the user has been signed up with Twitter you can access the id, name, screen_name or image and finally if a user has been signed up with Google+ you can access the id, name or image properties.

Users can have multiple authentication mechanisms. If a user is already authenticated and you authenticate it agaist Twitter, Facebook or Google+ authentication then the user is linked to these new providers. No new account is created, the already existing user is liked to those providers. If that's not the behaviour you are expecting just call `[Backbeam logout]` first.

## Where to go next

If you are using client-side business logic then you should learn: [how to mamipulate data](ios-data.md), [users authentication](ios-authentication.md), and [push notifications](ios-push-notifications.md).

If you think that server-side logic is better for your needs then you should learn [how to invoke the server-side logic from your iOS code](ios-server-side-logic.md).
