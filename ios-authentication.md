# Users authentication

This documentation is about authenticating users from your iOS code. This means you are using client-side logic and this requires you to create a pair of API keys and configure them in your `AppDelegate`. If you use server-side logic you should authenticate the user in the server-side, and the SDK will know when a signup/login/logout has happened. See the [Server-side logic section](ios-server-side-logic.md) for further information.

## Email + password based authentication

To register a user using email + password authentication just create an object indicating user as the object's entity, set the email and password and save the object.

```objectivec
BBObject* object = ...; // some object already created
[object setString:@"someone@example.com" forField:@"email"];
[object setString:@"your_strong_password" forField:@"password"];
[object save:^(BBObject* obj) {
  // Depending on your project settings the user is just logged in or is waiting to confirm their email address
  // You can check if the user is logged using [Backbeam currentUser]
} failure:^(BBObject* obj, NSError* error) {
  // Something went wrong
}];
```

If the user is already registered and wants to login you call the [Backbeam loginWithEmail:password:success:failure:] method.

```objectivec
[Backbeam loginWithEmail:@"user@example.com" password:@"the-password" success:^(BBObject *object) {
  // user is logged
} failure:^(NSError* error) {
  // Something went wrong
}];
```

You can optionally make some joins passing an optional argument before the callback.

If the user does not remember the password you can let them to request a password reset with the `[Backbeam requestPasswordResetWithEmail:success:failure:]` method. This will send the user an email with a link to a special web page (containing a verification code) to set a new password. But you can edit the email template to send the user to a custom webpage or include a link with a custom URL scheme containing the verification code. In this case you need to verify this code to authenticate the user.

```objectivec
[Backbeam verifyCode:code success:^(BBObject* user) {
  // code verified. You can now change the user's password, for example
} failure:^(NSError* error) {
  // Something went wrong
});
```

The method verifyCode also has an optional parameter before the callback to join some relationships of the user.

## Twitter authentication

To authenticate a user using Twitter you have at least two options: usin the Social framework introduced in iOS6 or using `BBTwitterSignupViewController`. In both cases the first thing you have to do is to configure the Twitter API keys. The best place to do so is in your AppDelegate:

```objectivec
#import "Backbeam.h"

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  [Backbeam setProject:@"your_project" sharedKey:@"your-shared-key" secretKey:@"your-secret-key" environment:@"dev"];
  [Backbeam setTwitterConsumerKey:@"..." consumerSecret:@"..."];

  // set up your application interface...
}
```

With the Social framework you first ask permission to the user to access their Twitter accounts. This is not enough to get the access tokens of the user, but luckily Backbeam implements the Twitter's reverse-oauth mechanism. Once you get the access tokens of a given account then you can signup the user (or send the tokens to the server).

```objectivec
//  You *MUST* keep the ACAccountStore alive for as long as you need an ACAccount instance
//  See WWDC 2011 Session 124 for more info.
self.accountStore = [[ACAccountStore alloc] init];

//  We only want to receive Twitter accounts
ACAccountType *twitterType = [self.accountStore accountTypeWithAccountTypeIdentifier:ACAccountTypeIdentifierTwitter];

//  Obtain the user's permission to access the store
[self.accountStore requestAccessToAccountsWithType:twitterType options:nil completion:^(BOOL granted, NSError *error) {

    // be sure to run this code in the main thread
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        if (error) {
            NSLog(@"error %@", error);
            return;
        }
        if (!granted) {
            NSLog(@"user canceled permission");
            return;
        }

        NSArray *accounts = [self.accountStore accountsWithAccountType:twitterType];

        // Give the user the option to choose an account
        ACAccount *account = accounts[0];

        NSLog(@"username: %@", account.username);

        // Calculate the account tokens using the reverse-oauth mechanism
        [Backbeam twitterReverseOAuthWithAccount:account success:^(NSDictionary *params) {

            // The dictionary contains the access tokens as well as the `screen_name` and `user_id`
            NSString* oauthToken       = [params objectForKey:@"oauth_token"];
            NSString* oauthTokenSecret = [params objectForKey:@"oauth_token_secret"];

            // Finally signup the user with the given tokens
            [Backbeam twitterSignupWithOAuthToken:oauthToken oauthTokenSecret:oauthTokenSecret success:^(BBObject *user, BOOL isNew) {
                NSLog(@"success!!");
            } failure:^(NSError *error) {
                NSLog(@"error %@", error);
            }];

        } failure:^(NSError *error) {
            NSLog(@"error %@", error);
        }];
    }];
}];
```

The downside of using the Social framework is that if the user doesn't have any Twitter account configured in the Configuration Settings, then he can't be able to sign in using this method. But the SDK provides an alternative method that is based in a UIWebView and implements the OAuth authentication flow. This functionality is implemented in the class named `BBTwitterSignupViewController`. This controller is very simple. You can open it inside a UINavigationController (and configure its navigationItem) or present it modally.

```objectivec
BBTwitterSignupViewController* vc = [Backbeam twitterSignupViewController];
// in this case the view-controller is pushed into a UINavigationController
[self.navigationController pushViewController:vc animated:YES];
[vc signup:^(BBObject* user, NSDictionary* extraInfo, BOOL isNew) {
  // The user is now logged.
  // Inside 'extraInfo' you can get the 'oauth_token' and 'oauth_token_secret'
  // if you need them to fetch further information of this user in Twitter
  // The 'isNew' flag indicates whether this user authenticated against your application
  // for the first time or is a returning user
  [self.navigationController popViewControllerAnimated:YES];
} failure:^(NSError* error) {
  NSLog(@"error %@", error);
} progress:^(BBSocialSignupProgress progress) {
  // Depending on the progress state you could show a HUD to the user like SVProgressHUD https://github.com/samvermette/SVProgressHUD
    if (progress == BBSocialSignupProgressLoadingAuthorizationPage) {
        NSLog(@"loading...");
    } else if (progress == BBSocialSignupProgressLoadedAuthorizationPage) {
        NSLog(@"loaded");
    } else if (progress == BBSocialSignupProgressAuthorizating) {
        NSLog(@"authorizating...");
    } else if (progress == BBSocialSignupProgressRedirecting) {
        NSLog(@"redirecting...");
    }
}];
```

The progress block is optional. There is even a version of this method without that parameter. But it is a good place to show some indicator to the user that there is something in progress.

The ViewController's UI is intentionally minimal. You can change the navigationItem, you could add an activity indicator, etc.

In order to use this mecanism you need to put some value in the callback parameter in your Twitter application. You can put anything. For example http://example.com. If you don't configure this parameter the WebView won't work. See [this discussion](https://dev.twitter.com/discussions/1250) that contains this message from the Twitter support:

> You need to have a placeholder URL on your app settings -- it doesn't matter what that URL is, really, but you should have one there. The presence of that URL allows you to perform callback-based authorization

## Facebook authentication

To support facebook authentication you first need to set up your application as explained in the official Facebook documentation. For example you must set a FacebookAppID key in your Info.plist file.

Then you will authenticate the user using the Facebook SDK and then you will pass the session's access_token to the Backbeam SDK. This is a quick example of how to get an access_token and pass it to the Backbeam SDK to authenticate a user.

```objectivec
NSArray *permissions = [NSArray arrayWithObjects:@"email", nil];
[FBSession openActiveSessionWithReadPermissions:permissions allowLoginUI:YES completionHandler:^(FBSession *session, FBSessionState status, NSError *error) {

  NSString *token = [[FBSession activeSession] accessTokenData].accessToken;
  if (token) {
    [Backbeam facebookSignupWithAccessToken:token success:^(BBObject* user, BOOL isNew) {
      // The user is authenticated in backbeam
      // The 'isNew' flag indicates whether this user authenticated against your application
      // for the first time or is a returning user
    } failure:^(NSError* err) {
      // Something went wrong
    }];
  } else {
    // check the facebook session status with the `status` variable or see if there is some `error`
  }
}];
```

Note that when the user wants to logout you should log them out from both the Backbeam SDK and the Facebook SDK. And you need to be aware that the Facebook SDK and the Bacbkeam SDK session information is not synchronized automatically. For example the Facebook session could expire, but Backbeam sessions do not expire. This is how you can logout the user from Backbeam and from Facebook as well:

```objectivec
[Backbeam logout];
[[FBSession activeSession] closeAndClearTokenInformation];
```

## Google+ authentication

First of all you should install the Google+ SDK and configure your application as explained in the official website: Google+ Sign-In for iOS. You can stop at the Add the sign-in button section. Then you will have a code similar to this:

```objectivec
GPPSignIn *signIn = [GPPSignIn sharedInstance];
signIn.clientID = YOUR_GOOGLE_PLUS_CLIENT_ID;
signIn.scopes = @[kGTLAuthScopePlusLogin]; // defined in GTLPlusConstants.h
signIn.delegate = self;

[signIn authenticate];
```

When the user completes the authorization step successfully your delegate method will be called and then you can use the accessToken to signup the user into Backbeam. Remember to have enabled in your Google APIs console the Google+ API service in the Services section.

```objectivec
- (void)finishedWithAuth:(GTMOAuth2Authentication *)auth error:(NSError *)error {

    [Backbeam googlePlusSignupWithAccessToken:auth.accessToken success:^(BBObject *user, BOOL isNew) {
    // The user is authenticated in backbeam
    // The 'isNew' flag indicates whether this user authenticated against your application
    // for the first time or is a returning user
    } failure:^(NSError *error) {
        // Something went wrong
    }];
}
```

If you get a redirect_uri_mismatch error probably means that you haven't set correctly the Bundle ID in the Google API console. It must match the one in your project settings in XCode. If you get a `[__NSDictionaryM gtm_httpArgumentsString]: unrecognized selector sent to instance` error in XCode4 probably you need to add the -objC flag in the Other Linker Flags setting inside Build settings in your XCode project.

Note that when the user wants to logout you should log them out from both the Backbeam SDK and the Google+ SDK. And you need to be aware that the Google+ SDK and the Bacbkeam SDK session information is not synchronized automatically. For example the Google+ session could expire, but Backbeam sessions do not expire. This is how you can logout the user from Backbeam and from Google+ as well:

```objectivec
[Backbeam logout];
[[GPPSignIn sharedInstance] disconnect];
```

## LinkedIn authentication

In order to use LinkedIn authentication you use the `linkedInSignupWithAccessToken:success:failure` method in the Backbeam class. This requires you to obtain the access token of the user by yourself. But additionally the SDK comes with a helper ViewController with a WebView to handle all the Oauth2 implementation.

You need to configure the project first. A good way to do so is in your AppDelegate. You need to create a LinkedIn app and pass the clientId and clientSecret to the Backbeam SDK:

```objectivec
[Backbeam setLinkedInClientId:@"..." clientSecret:@"..."];
Then at any time you create a ViewController that contains a UIWebView that will guide the user in the authentication steps. This is an example of use.

BBLinkedInSignupViewController *vc = [Backbeam linkedInSignupViewController];
[self.navigationController pushViewController:vc animated:YES];
[vc signup:^(BBObject *user, NSDictionary *extraInfo, BOOL isNew) {
    NSLog(@"success %@ extra %@", [user linkedInData:@"headline"], extraInfo);
    [self.navigationController popViewControllerAnimated:YES];
} failure:^(NSError *err) {
    NSLog(@"error %@", err);
} progress:^(BBSocialSignupProgress progress) {
    if (progress == BBSocialSignupProgressLoadingAuthorizationPage) {
        NSLog(@"loading...");
    } else if (progress == BBSocialSignupProgressLoadedAuthorizationPage) {
        NSLog(@"loaded");
    } else if (progress == BBSocialSignupProgressAuthorizating) {
        NSLog(@"authorizating...");
    } else if (progress == BBSocialSignupProgressRedirecting) {
        NSLog(@"redirecting...");
    }
}];
```

The ViewController's UI is minimal intentionally. You can use the progress block to update the interface adding an activity indicator for example between the "loading" and "loaded" steps. You can also modify the navigationItem of the view controller to put buttons, etc. If you want to use the LinkedIn API you can access the access_token in the extraInfo dictionary returned when the signup is successful.

## GitHub authentication

In order to use GitHub authentication you use the `gitHubSignupWithAccessToken:success:failure` method in the Backbeam class. This requires you to obtain the access token of the user by yourself. But additionally the SDK comes with a helper ViewController with a WebView to handle all the Oauth2 implementation.

You need to configure the project first. A good way to do so is in your `AppDelegate`. You need to register a GitHub application and pass the clientId, clientSecret and authorization callback URL to the Backbeam SDK:

```objectivec
[Backbeam setGitHubClientId:@"..." clientSecret:@"..." callbackURL:@"..."];
```

Actually the callback URL won't be loaded at any moment if everything goes well, but the GitHub's OAuth implementation requires to pass the URL in the process.

Once you have your project configured at any time you create a ViewController that contains a `UIWebView` that will guide the user in the authentication steps. This is an example of use.

```objectivec
BBGitHubSignupViewController *vc = [Backbeam gitHubSignupViewController];
[self.navigationController pushViewController:vc animated:YES];
[vc signup:^(BBObject *user, NSDictionary *extraInfo, BOOL isNew) {
    NSLog(@"success %@ extra %@", [user gitHubData:@"login"], extraInfo);
    [self.navigationController popViewControllerAnimated:YES];
} failure:^(NSError *err) {
    NSLog(@"error %@", err);
} progress:^(BBSocialSignupProgress progress) {
    if (progress == BBSocialSignupProgressLoadingAuthorizationPage) {
        NSLog(@"loading...");
    } else if (progress == BBSocialSignupProgressLoadedAuthorizationPage) {
        NSLog(@"loaded");
    } else if (progress == BBSocialSignupProgressAuthorizating) {
        NSLog(@"authorizating...");
    } else if (progress == BBSocialSignupProgressRedirecting) {
        NSLog(@"redirecting...");
    }
}];
```

The ViewController's UI is minimal intentionally. You can use the progress block to update the interface adding an activity indicator for example between the "loading" and "loaded" steps. You can also modify the navigationItem of the view controller to put buttons, etc. If you want to use the LinkedIn API you can access the `access_token` in the `extraInfo` dictionary returned when the signup is successful.
