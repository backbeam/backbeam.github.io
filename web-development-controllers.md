# Controllers

All request are routed and executed using controllers. When you create a new controller you specify the HTTP method that this controller will handle and the path-pattern. The most simple route would be an HTTP GET method responding to the / (root) path.

Controllers are written in JavaScript with the JavaScript SDK plus some additions specific of web development.

The most simple controller would be a controller with this code:

```javascript
response.send('Hello world')
```

You edit the code right in your browser. There is no need to use any other development tool. You will see two URLs below the code editor similar to this: `http://web-v1- dev-your-project.backbeamapps.com/`. By default Backbeam creates a few subdomains for your project (this is explained in the next section). You can access that URL right now. While you are developing the URLs are password-protected. You can see the credentials you need to use in the "Web development" section. This way your web is only visible to the world when it is ready.


## URL patterns

When you create a controller you can define a constant path (such as `/users/top-users-in-my-webapp`) or a pattern. Patterns contain path segments beginning with : and followed by a name. You will be able to access the value of that path segment using that name in your controller. For example if you define a URL pattern like this:

`/items/:item`

If someone requests the path `/items/some-item` then you can access the parameter called 'item' in your code as follows:

```javascript
// request.params.item = 'some-item' in the example (/items/some-item)
response.send('Requesting item: '+request.params.item)
```
