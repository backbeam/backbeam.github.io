# Server side logic

This section explains how to use server-side logic from your Android code. You don't need API keys for this functionality. If you only use server-side logic you don't have to set the API keys in your main activity but you need to configure the `webVersion` and the `httpAuth` settings.

## Requesting objects from controllers

If your controller "sends objects" you can invoke that controller from your iOS code like in the following example:

```java
Backbeam.requestObjectsFromController("GET", "/", null, FetchPolicy.REMOTE_ONLY, new FetchCallback() {
  @Override
  public void success(List<BackbeamObject> objects, int totalCount, boolean fromCache) {
    System.out.println("Success!! "+objects.size());
  }
});
```

The SDK will tell the controller the current logged user (if there is someone logged). And if the controller makes some authentication stuff it will tell the SDK if other user has logged in or the current user has logged out. If the authenticated user changes and the object that represents that user is not present in the response then the `Backbeam.currentUser()` will change but it will be empty.

## Requesting JSON from controllers

You can also invoke controllers that send JSON manually. The same side-effects related to the users authentication apply so the information of the current authenticated user is exchanged between the SDK and the controller. This is an example of how to invoke a controller that just sends JSON

```java
Backbeam.requestJsonFromController("GET", "/", null, FetchPolicy.REMOTE_ONLY, new RequestCallback() {
  @Override
  public void success(Json json, boolean fromCache) {
    System.out.println("Success!! "+json);
  }
});
```

The only downside is that if the authenticated user changes the SDK will be notified and the new `Backbeam.currentUser()` will have changed but it will be empty. You could refresh it to have all its values populated.

## Uploading and downloading data

To upload files to a controller you can use the `FileUpload` class. There are two ways of creating a `FileUpload`:

```java
new FileUpload(File file, String mimeType)
new FileUpload(InputStream inputStream, String filename, String mimeType)
```

Once you have created a `FileUpload` you can pass it in the `params` argument when invoking a web controller. For example:

```java
InputStream stream = null;
TreeMap<String, Object> params = new TreeMap<String, Object>();
params.put("file", new FileUpload(stream, "picture.jpg", "image/jpeg"));
Backbeam.requestJsonFromController("POST", "/upload", params, FetchPolicy.REMOTE_ONLY, new RequestCallback() {

	@Override
	public void success(Json json, boolean fromCache) {
		System.out.println("Success!! "+json);
	}

	@Override
	public void failure(BackbeamException exception) {
		System.out.println("exception");
		exception.printStackTrace();
	}
});
```

In your web controller you will be able to acces the file using:

```javascript
var file = request.files['file']
```

For further information about how to handle files in server-side code refer to the section about [Save uploaded objects](javascript-server-side-logic.html#save-uploaded-objects)
