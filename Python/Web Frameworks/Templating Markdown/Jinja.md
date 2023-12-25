# Jinja Templating Basics
Jinja is a powerful templating engine used in Flask and other Python web frameworks. It allows you to generate dynamic content by combining templates with data to produce the final output, such as HTML pages.

Can be used with arguments to render templates:
> See  [[09 Rendering Templates and Static Files#Rendering Templates]]

## Template Syntax
Jinja uses double curly braces `{{ }}` for variable interpolation. You can place variables inside the curly braces to dynamically render their values. For example: `{{ name }}`.

## Control Structures
Jinja provides control structures like `if` statements and loops to control the flow of template rendering. You can use `{% %}` tags to enclose control structures. 

If else conditions are represented like so:
```django
{% if condition %}
<!-- code to execute when condition is true -->
{% else %}
<!-- code to execute when condition is false -->
{% endif %}
```

For loops are represented like so: 
```django
{% for x in y %}
<div> ... </div> <!-- HTML to display on each iteration -->
{% endfor %}
```


## Template Inheritance
Jinja supports template inheritance, allowing you to define a base template with common elements and extend it in child templates to override or add specific content. This promotes code reuse and provides a clean structure for your templates.

Base template (`base.html`):
```django
<html>
<head>
    <title>{% block title %}My Website{% endblock %}</title>
</head>
<body>
    {% block content %}{% endblock %}
</body>
</html>
```

Child template (`child.html`):
```django
{% extends "base.html" %}

{% block title %}Welcome to My Website{% endblock %}

{% block content %}
    <h1>Welcome</h1>
    <p>Thank you for visiting our website!</p>
{% endblock %}
```

Here `{% extends "base.html" %}` tells Jinja that this template should replace the block from the base template called `title` with the first block and the block in the base template called `content` with the second block in `child.html`

## Filters
Jinja provides filters to modify or format data within the template. Filters are applied to variables using the pipe symbol `|`. For example: `{{ name|capitalize }}`

# Forms
Jinja supports forms which when dispatched hold the form data in the flask `request.form` object
- The `method` attribute sets the request type on submit
```html
<form method="post">
  <label for="username">Username</label>
  <input name="username" id="username" required />
  <label for="password">Password</label>
  <input type="password" name="password" id="password" required />
  <input type="submit" value="Log In" />
</form>
```
When this form is submitted, the data will be dispatched through a `post` to the route responsible for rendering this template using `render_template()` (See  [[09 Rendering Templates and Static Files#Rendering Templates]])

We can also pass URL parameters through Jinja forms using [[Python/Web Frameworks/Flask/03 Routing#URL Building]] which can be accessed through [[Python/Web Frameworks/Flask/05 Accessing Request Data#URL Parameters]]
```html
 <a class="action" href="{{ url_for('blog.update', id=post['id']) }}">Edit</a>
```
