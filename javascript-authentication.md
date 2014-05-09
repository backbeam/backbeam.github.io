# Users authentication

## Session storage

When you perform authentication functionality you can access the current authenticated user with var user = `backbeam.currentUser()`. When you are developing a web page the user session is stored in a cookie. If you are using the SDK in the client-side (i.e an embedded mobile application) then by default the SDK only remembers the current authenticated user saving the object in memory. But you can also use a persistent session storage. You just need to pass an optional parameter to the backbeam.configure method and implement two functions. This is an example of how to implement a basic file-based session persistence in Nodejs:

```javascript
var sessFile = require('path').join(__dirname, 'session-data')
backbeam.configure({…,
  sessionStore: {
    store: function(sessionData) {
      if (sessionData) { // sessionData is just a string. If it is null it is due a logout
        fs.writeFileSync(sessFile, sessionData, 'utf8')
      } else {
        fs.unlinkSync(sessFile)
      }
    },
    restore: function() {
      try {
        return fs.readFileSync(sessFile, 'utf8')
      } catch(e) {
        return null
      }
    }
  }
})
```

## Email + password based authentication

For a simple signup using email+password you just need to create a user object and save it with its email and password fields setted. You of course can set more fields.

```javascript
var object = backbeam.empty('user')
object.set('email', 'some_user@example.com')
object.set('password', 'some_strong_password')
object.save(function(error) {
  // here the user has been created
  var user = backbeam.currentUser()
  // if user != null both 'user' and 'object' are the same object
})
```

Depending on your project settings the user is registered and can logged or just registered waiting to activate their email address. If `backbeam.currentUser()` does not return null the user is both registered and logged in the application.

`backbeam.currentUser()` is persistent in server-side code. The current user is stored in a cookie and you can access it in any controller. You can logout the user by calling `backbeam.logout()`.

When you want to authenticate an already registered user using email+password authentication you just need to use `backbeam.login()` as follows

```javascript
backbeam.login('some_user@example.com', 'some_strong_password', function(error, user) {
  // if there is no error you can access the users information
})
```

For server-side code once the user has been authenticated you can access the `backbeam.currentUser()` at anytime in any controlller until you call `backbeam.logout()` or the cookie expires.

## Third-party authentication

You can authenticate your users using Twitter, Facebook, Google+, LinkedIn and GitHub. First of all you need to configure your project: go to the 'Users authentication' in your control panel and configure the Twitter and Facebook parameters.

In the JavaScript SDK you get authorization of your users and then you send to Backbeam their credentials. For Twitter the credentials consist on the `oauth_token` and `oath_token_secret` generated in the OAuth authorization process.

```javascript
var credentials = { oauth_token:'the oauth token', oauth_token_secret:'the oauth token secret' }
backbeam.twitterSignup(credentials, function(err, user, isNew) {
  // Here you get the 'user' object if everything went ok
  // The 'isNew' boolean indicates whether the user is new or is a returning user
})
```

When developing web applications backbeam provides some helper methods for the Twitter authentication. Read [how to signup users using Twitter in your web application](http://backbeam.io/article/signup-users-using-twitter-in-your-web-application).

When using Facebook as authentication mechanism you will get an `access_token` and then you can signup the user into your project using:

```javascript
var credentials = { access_token:'the acccess token' }
backbeam.facebookSignup(credentials, function(err, user, isNew) {
  // Here you get the 'user' object if everything went ok
  // The 'isNew' boolean indicates whether the user is new or is a returning user
})
```

If you want to sign up a user from a web application using Facebook you can use the documented [Login flow in the Facebook documentation](https://developers.facebook.com/docs/facebook-login/login-flow-for-web-no-jssdk/).

The process for Google+, LinkedIn and GitHub are very similar:

```javascript
var credentials = { access_token:'the acccess token' }
backbeam.googlePlusSignup(credentials, function(err, user, isNew) {
  // Here you get the 'user' object if everything went ok
  // The 'isNew' boolean indicates whether the user is new or is a returning user
})
```

For Google+ be sure to have enabled the Google+ API service in your project in the Google APIs console or the signup will fail.

For LinkedIn you need to use the linkedInSignup method and pass an object with an `access_token` field. You will need to create first a [LinkedIn app](https://www.linkedin.com/secure/developer) and use some implementation of the OAuth2 protocol to get the access_token.

For GitHub you need to use the gitHubSignup method and pass an object with an `access_token` field. You will need to register a [GitHub application](https://github.com/settings/applications) and use some implementation of the OAuth2 protocol to get the access_token.
