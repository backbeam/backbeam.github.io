# Users authentication

This documentation is about authenticating users from your Android code. This requires you to create a pair of API keys and configure them in your main activity. If you use server-side logic you should authenticate the user in the server-side, and the SDK will know when a signup/login/logout has happened. See the [Server-side logic section](android-server-side-logic.md) for further information.

## Email+password authentication

If you want to register a user using email+password based authentication you just create an object member of the "user" entity, set its email and password and save it. Depending on your project configuration the user will be just registered or will wait fot the user to confirm their email address.

```java
BackbeamObject obj = new BackbeamObject("user");
obj.setString("email", "user@example.com");
obj.setString("password", "the-password");
obj.save(new ObjectCallback() {
    @Override
    public void success(BackbeamObject user) {
        BackbeamObject currentUser = Backbeam.currentUser();
        // if currentUser != null it means that he/she is registered
        // if currentUser == null it means that he/she needs to confirm the email address
    }
});
````

If the user is already registered and wants to login you call the Backbeam.login method.

```java
Backbeam.login(email, password, new ObjectCallback() {
    @Override
    public void success(BackbeamObject user) {
        // successful login
    }
});
```

You can optionally make some joins passing an optional argument before the callback.

If the user does not remember the password you can let them to request a password reset with the `Backbeam.requestPasswordReset` method. This will send the user an email with a link to a special web page (containing a verification code) to set a new password. But you can edit the email template to send the user to a custom webpage or include a link with a custom URL scheme containing the verification code. In this case you need to verify this code to authenticate the user.

```java
Backbeam.verifyCode(code, new ObjectCallback() {
    @Override
    public void success(BackbeamObject user) {
        // code verified. You can now change the user's password, for example
    }
});
```

The method `verifyCode` also has an optional parameter before the callback to join some relationships of the user.

```java
Backbeam.verifyCode(code, "join favorite-place", callback);
```

## Twitter authentication

In order to authenticate a user with Twitter you first need to request the user grant access to their Twitter profile using a OAuth library or other mechanism. At the end you will have two tokens: an `oauth_token` and an `oauth_token_secret`. Then you can pass them to Backbeam to signup the user

```java
Backbeam.twitterSignup(oauthToken, oauthTokenSecret, new SignupCallback() {

  @Override
  public void success(BackbeamObject user, boolean isNew) {
    // `user` is the object that represents the user and `isNew` tells you whether the user is a returning user or not
  }
});
```

## Facebook authentication

In order to authenticate a user with Facebook you first need to install the Facebook SDK and request access to give you access to his account. Then you will be able to access the accessToken of the session using `getAccessToken()`.

```java
Backbeam.facebookSignup(session.getAccessToken(), new SignupCallback() {

  @Override
  public void success(BackbeamObject user, boolean isNew) {
    // `user` is the object that represents the user and `isNew` tells you whether the user is a returning user or not

  }
});
```

## Google+ authentication

To allow users authenticate using Google+ you need to follow the steps in the Google+ Sign-in for Android. Remember to have enabled in your Google APIs console the Google+ API service in the Services section.

Once you have the authenticated user

```java
Backbeam.googlePlusSignup(accessToken, new SignupCallback() {

  @Override
  public void success(BackbeamObject user, boolean isNew) {
    // `user` is the object that represents the user and `isNew` tells you whether the user is a returning user or not

  }
});
```

## LinkedIn authentication

To allow users authenticate using LinkedIn you need to create a LinkedIn app and use some implementation of the Oauth2 protocol to get a valid `access_token`.

Once you have the authenticated user

```java
Backbeam.linkedInSignup(accessToken, new SignupCallback() {

  @Override
  public void success(BackbeamObject user, boolean isNew) {
    // `user` is the object that represents the user and `isNew` tells you whether the user is a returning user or not

  }
});
```

## GitHub authentication

To allow users authenticate using LinkedIn you need to register a GitHub application and use some implementation of the Oauth2 protocol to get a valid access_token.

Once you have the authenticated user

```java
Backbeam.gitHubSignup(accessToken, new SignupCallback() {

  @Override
  public void success(BackbeamObject user, boolean isNew) {
    // `user` is the object that represents the user and `isNew` tells you whether the user is a returning user or not

  }
});
```

## Anonymous users

In some circumstances you could need to start grabbing information of the user (such as preferences, etc.) before it is actually registered. Or maybe you don't want the user to either choose a password or connect with Facebook or Twitter. In these cases you just create an object of type "user" and save it. You automatically have the `Backbeam.currentUser()` available. And at any moment you can set an email+password or link the user to a Facebook or Twitter account (or both!).
