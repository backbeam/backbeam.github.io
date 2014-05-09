# Server side logic

This section explains how to use server-side logic from your iOS code. You don't need API keys for this functionality. If you only use server-side logic you can set your API keys to nil in your `AppDelegate` but you need to configure the `webVersion` and the `httpAuth` settings.

## Requesting objects from controllers

If your controller "sends objects" (see Sending objects from controllers) you can invoke that controller from your iOS code like in the following example:

```objectivec
[Backbeam requestObjectsFromController:@"/" method:@"GET" params:nil fetchPolicy:BBFetchPolicyRemoteOnly success:^(NSArray *objects, NSInteger totalCount, NSArray *distances, BOOL fromCache) {
    // Now do whatever you want with the array of BBObjects
} failure:^(NSError *error) {
    NSLog(@"error %@", error);
}];
```

The SDK will tell the controller the current logged user (if there is someone logged). And if the controller makes some authentication stuff it will tell the SDK if other user has logged in or the current user has logged out. If the authenticated user changes and the object that represents that user is not present in the response then the [Backbeam currentUser] will change but it will be empty.

## Requesting JSON from controllers

You can also invoke controllers that send JSON manually. The same side-effects related to the users authentication apply so the information of the current authenticated user is exchanged between the SDK and the controller. This is an example of how to invoke a controller that just sends JSON

```objectivec
[Backbeam requestJSONFromController:@"/" method:@"GET" params:nil fetchPolicy:BBFetchPolicyRemoteOnly success:^(id result, BOOL fromCache) {
    // `result` can be a NSDictionary or an NSArray depending what the controller sent in the response
} failure:^(id result, NSError *error) {
    NSLog(@"error %@", error);
}];
```

The only downside is that if the authenticated user changes the SDK will be notified and the new [Backbeam currentUser] will have changed but it will be empty. You could refresh it to have all its values populated.

## Uploading and downloading data

These methods have an optional progress argument to listen to the upload progress. This is useful when uploading data like an image for example. To upload files to a controller you can use the BBFileUpload class. This is an example of how to invoke a controller with a file upload and listen to the upload progress to update a UIProgressView.

```objectivec
// Conver an image to an NSData object
NSData *data = UIImageJPEGRepresentation(image, 0.7);

// Create the fileUpload object
BBFileUpload *fileUpload = [[BBFileUpload alloc] initWithData:data
                                                     mimeType:@"image/jpg"
                                                     fileName:@"picture.jpg"];

NSDictionary *params = [NSDictionary dictionaryWithObjectsAndKeys:fileUpload, @"file", nil];
// `file` is the name of the param that the controller will use: var file = request.files['file']

[Backbeam requestJSONFromController:@"/change-user-picture" method:@"POST" params:params fetchPolicy:BBFetchPolicyRemoteOnly progress:^(NSInteger lastBytesSentCount, long long sentBytes, long long totalBytes) {
    [self.progressView setProgress:(totalBytes/sentBytes) animated:YES];
} success:^(id result, BOOL fromCache) {
    NSLog(@"success %@", result);
} failure:^(id result, NSError *error) {
    NSLog(@"error %@", error);
}];
```

Finally there is also a method that lets you download raw data from a controller and lets you listen to both: the upload and download progress. This method is as follows:

```objectivec
+ (void)requestDataFromController:(NSString*)path
                           method:(NSString*)method
                           params:(NSDictionary*)params
                      fetchPolicy:(BBFetchPolicy)fetchPolicy
                   uploadProgress:(ProgressDataBlock)uploadProgress
                 downloadProgress:(ProgressDataBlock)downloadProgress
                          success:(SuccessDataBlock)success
                          failure:(FailureBlock)failure
```

