# Features overview

Backbeam offers many services that you will use to build your web and mobile applications. Backbeam is also designed to help you in your development process, not only when your application is finished.

## Environments

Every Backbeam project has two environments. The development environment and the production environment.

Both environments share the same database schema / structure, but store different data. There are other differences between them that you will discover. For example for iOS you can have different push notification certificates in each environment.

## Database

On Backbeam you define your data model to fit your needs. You create entities and fields (like you create tables and columns in a database) and you can also define relationships between entities.

The development and production environments share the same data model. This way your code will always behave the same in both environments.

Nevertheless you have to be sure of what you want to do when making destructive changes to your data model such as deleting fields, relationships or entities. Be careful also if you rename fields.

## Control panel

The control panel is the web interface of your project. In the control panel you define your data model, you are able to browse your data, configure other services such as push notifications and email delivery and you can invate teammates to your project.

If you write server-side code (further information on [Where to write the business logic](overview-where-to-write-the-business-logic.md) the control panel will also be your IDE. You can write the server-side logic of a mobile app or you can write a full featured web application or website.
