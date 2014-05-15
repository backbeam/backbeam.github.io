# Users authentication

Backbeam has builtin functionality to authenticate your users. By default your database has a `user` entity. This is the entity you will use to administrate your users.

## Authentication methods

Backbeam comes with a few authentication mechanisms.

### Email + password

Backbeam implements the classic email+password scheme for users authentication. But it does not just implement the secure password storage. It also implements the confirmation step and the *lost password* functionality if you want to use it.

> Passwords are not stored directly in the database. Backbeam stores the result of a few cryptographic computations including the use of HMAC and an adaptive hashing function.
> This protects the service against [rainbow table attacks](http://en.wikipedia.org/wiki/Rainbow_table), brute force attacks and [length extension attacks](http://en.wikipedia.org/wiki/Length_extension_attack)

Backbeam comes with default email templates to confirm the email address of the user but you can personalize them completely. You only need to configure an email delivery service (SMTP, Amazon or Postmark) in the control panel.

You can make the email confirmation mandatory, optional, or avoid any email confirmation. This can be configured in the *Configuration* section of your project.

### Third party authentication methods

Backbeam can also authenticate your users using third party services such as Facebook, Twitter, Google+, LinkedIn and GitHub.

Backbeam offers APIs and methods to verify the identity of the user using tokens (usually OAuth tokens) returned by these services after their authentication steps. Some SDKs offer more functionality implementing the full authentication process against these services, but in some situations you will need to implement the steps needed to obtain the authentication tokens. For example for Facebook integrations you will use the SDKs that Facebook provides, you will get an `access_token` string, and you will pass that string to Backbeam to identify the user. Backbeam will create a new user if this is the first time the user signs in, or you will receive the `user` instance that corresponds to that identity if the user alredy signed in previously.

## Multiple authentication mechanisms

On Backbeam a user can have multiple authentication mechanisms at the same time. Just authenticate the user with one mechanism and while he is logged in, authenticate him using other mechanisms. The user then will have all the identities tied to the same `user` object.

## Anonymous users

You can also create anonymous users. You just create an empty `user` object and that's it. The SDK you are using will keep the session of the user and you can store information on it, or in other relationed entities.

The only downside is that that user won't log in into your app again if for any reason he loses the session. But this is a useful functionality if you want the user to be able to use the service and eventually add authentication mechanisms such as a password.

#### Feedback

Is there any other service you want to be implemented in Backbeam? Leave a comment in that case :)
