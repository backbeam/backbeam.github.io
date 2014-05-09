# Subdomains and custom domain

Backbeam creates a few subdomains for your web application. The pattern is:

`http://web-[<version>-]<env>-<your-project>.backbeamapps.com`

The first subdomain is optional and defines the web version to use. Each web version has an identifier (v1, v2,...). If you omit this parameter the default version is used. You can change the default version in the "Views and controllers" section of your project. The second subdomain is the environment to use which is either `dev` or `pro`. Depending on this parameter you will access either the development or the production database.

When using a custom domain it will point to the default version at the moment in the pro environment. To configure your custom domain you need to go to your project configuration and define it. Then you need to change your DNS configuration creating a `CNAME` record directed to `backbeamapps.com`.

