# Views (templates)

Usually you won't send HTML to the browser using `response.send('<html>...</html>')`. You can use views instead. This feture lets you create templates with a powerful templating engine to generate dynamic HTML (or any other text-based format).

This templating engine is called [nunjucks](http://mozilla.github.io/nunjucks/) and it's maintained by Mozilla.

You can create for example an `index.html` view and write

```html
<html><h1>Hello world!</h1></html>
```

Then in your controller can render this view just calling:

```javascript
response.render('index.html')
```

But you can do more than that. You can pass options options to the template and inside the template you can do if statements, loops, print values,... Let's see a simple yet more advanced example. This could be your view

```html
<html><h1>Hello {{ message }}!</h1></html>
```

This could be your controller:

```javascript
response.render('index.html', { message: 'from Backbeam' })
```

Let's see an example using data from the database:

```javascript
// supposing we have an 'item' entity with a 'name' field
backbeam.select('item').query('sort by created_at asc').fetch(100, 0, function(err, items) {
    response.render('index.html', { items: items }, true)
    // the third argument set to 'true' means that the output will be autoescaped
})
```

This could be your view:

```html
<html>
<body>
    <ul>
    {% for item in items %}
        <li>{{ item.get('name') }}</li>
    {% endfor %}
    </ul>
</body>
</html>
```

Note that views are only accessible from controllers, they are not accessible directly through URLs.

Remember that you can learn more on how to use controllers reading the JavaScript SDK documentation.

## Autoescaping

The response.render() method accepts optionally a boolean third argument. If set to true tells the template engine to autoescape the output. So if you have a controller like this:

```javascript
var message = 'Hello <strong>world</strong>'
response.render('index.html', { message:message }, true)
```

And you want to print the message in a template like this:

`Message: {{ message }}`

The characters of the message will be escaped so you won't see anything in a bold font. This prevents Cross-site-scripting attacks. If you aren't using autoescaping you should escape manually the output as follows:

`Message: {{ message|escape }}`

Sometimes you don't want some values to be escaped. For example if you want to show the content of a rich-text field. If autoescaping is activated you can prevent a certain value to be escaped using the safe filter:

`Message: {{ message|safe }}`

## Custom filters
You can extend the capabilities of the templating engine by setting your own custom filters. There are several builtin filters, for example:

`{{ my_string_variable|default('the string was empty') }}`

But you can also define your own filters. You need to create a file called `filters.js` in the Libraries section. All the functions exported in this file will be used as custom filters. For example, if you define a function like this in the filters.js file:

```javascript
module.exports.pluralize = function(count, singular, plural) {
    return count > 1 ? plural : singular
}
```

You can just start using it in your templates. For example:

```html
{% set count = 1 %}
<p>{{ count }} {{ count|pluralize('category', 'categories') }}</p>

{% set count = 5 %}
<p>{{ count }} {{ count|pluralize('category', 'categories') }}</p>
```

The first parameter passed to the function is the variable to which the filter is applied. Then you can add more parameters as needed.
