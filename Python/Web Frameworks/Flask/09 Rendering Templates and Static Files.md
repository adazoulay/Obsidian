## Static files
Dynamic web applications also need static files.
* That’s usually where the CSS and JavaScript files are coming from
* Should be done by web server but Flask can also do it

To do so, create a folder called `static` , should be available at `/static` on the application
To generate URLs for static files, we can use the `static` endpoint name
(see [[Python/Web Frameworks/Flask/03 Routing#URL Building|URL Building]]):
```html
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}" />
```
The file has to be stored on the filesystem as `static/style.css`.

## Rendering Templates
With Flask, [[Jinja]] template engine comes builtin
To render a template you can use the `render_template()`  method
- All you have to do is provide the name of the template and variables you want to pass as keyword arguments
``` python
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', name=name)
```

Flask will look for templates in the `templates` folder.
- If your application is a module, this folder is next to that module:
 ```
/application.py
	/templates
		/hello.html
```
- if it’s a package it’s actually inside your package:
```
/application
    /__init__.py
    /templates
        /hello.html
```

Here is an example [[Jinja]] template:
```html
<!doctype html>
<title>Hello from Flask</title>
{% if name %}
  <h1>Hello {{ name }}!</h1>
{% else %}
  <h1>Hello, World!</h1>
{% endif %}
```
Inside templates you also have access to the `config`, `request`, `session` and `g` (see [[11 Design Patterns|Design Patterns]])  objects as well as the `url_for()` (see [[Python/Web Frameworks/Flask/03 Routing#URL Building|URL Building]]) and `get_flashed_messages()` functions


