# Templating engine

For some features Backbeam uses a templating engine called [nunjucks](http://mozilla.github.io/nunjucks/) and it's maintained by Mozilla.

This templating engine is used in email templates as well as in HTML templates for web development.

For a full reference of this templating engine check the [official documentation](http://mozilla.github.io/nunjucks/templating.html). But here you can see how to do the most basic things:

## Print data

You can print a variable value using

```
{{ username }}
```

If that variable is an object you can print its attributes using:

```
{{ foo.bar }}
{{ foo["bar"] }}
```

## Conditional statements

You can use `if-elif-else` to make conditional statements. Example:

```
{% if hungry %}
  I am hungry
{% elif tired %}
  I am tired
{% else %}
  I am good!
{% endif %}
```

## Loops

You can iterate arrays. For example:

```
<h1>Posts</h1>
<ul>
{% for item in myarray %}
  <li>{{ item.title }}</li>
{% endfor %}
</ul>
```

And you can iterate hashes/objects/dictionaries as well:

```
{% for key, value in object %}
  The property {{ key }} has a value of {{ value }}
{% endfor %}
```
