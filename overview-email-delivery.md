# Email templates and delivery

Email delivery is an optional service. It is used if you use email+password authentication with email verification but you can also deliver emails at any situation from the server-side code.

## Supported services

Backbeam does not deliver email directly. You can configure your project to use either Amazon SES, Postmark or any SMTP server.

Go to the *Configuration* section in the control panel and configure one of the available services.

## Email templates

Backbeam provides a few email templates that you can personalize that are needed for the email+password authentication mechanism.

Any email template is composed by three parts:

* The subject
* The HTML version
* The plain text version

The three parts use the [templating engine](overview-tempalting-engine.md) builtin on Backbeam.

In the *Configuration* section of your project you can choose to send email messages in HTML, plain text or both.

## Custom templates

You can create your own email templates. And you can send email messages using server-side logic. Check the *Sending emails* section on the [server-side code](javascript-server-side-logic.md) documentation for further information.
