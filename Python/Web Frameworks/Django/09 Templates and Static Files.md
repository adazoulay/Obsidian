# Templates
We can use Django’s template system to separate the design from Python by creating a template that our [[07 Views|view]] can use

We start by creating a `templates` directory in our app directory
- Django will automatically look for templates there

The project's `TEMPLATES` [[02 Setup and Settings.py#`Settings.py`|Settings]] describe how Django will load and render templates

>[!tip]
>Because of how `app_directories` template loader works, we need to create a sub directory in our `app_name/templates` folder called `app_name`
>Hence, the template should be at 
>`app_name/templates/app_name/index.html`
>We could just put `index.html` in `templates` but might cause naming conflict with multiple apps. This is just a name-spacing thing
>Because of how the `AppDirectoriesFinder` static-file finder works, you can refer to this static file in Django as `app_name/index.html`


Here is an example markdown template file. The syntax is very similar to [[Jinja]]
```django
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

### Removing hardcoded URLs in Templates
The problem with hardcoded urls are that they are tightly coupled

However, since we defined the name argument in the `path()` functions in the `myapp.urls` module (see [[Python/Web Frameworks/Django/03 Routing#`path()`|path()]]) we can use the  `{% url %}` template tag:

Instead of:
```django
<a href="/polls/{{ question.id }}/">{{ question.question_text }}</a>
```
We could have:
```django
<a href="{% url 'detail' question.id %}">{{ question.question_text }}</a>
```

`{% url %}` works by looking up the URL definition `name` parameter as specified in the `polls.urls` module (see [[Python/Web Frameworks/Django/03 Routing#`path()`|path()]])

```python
# the 'name' value as called by the {% url %} template tag
path("<int:question_id>/", views.detail, name="detail"),
```

This will match the URL for a [[07 Views|view]]  method, very similar to Flask's [[Python/Web Frameworks/Flask/03 Routing#URL Building|URL Building]] method `url_for()`

## Forms 
Forms can be used to query our backend in the same way as [[Jinja#Forms|Jinja Forms]] and the data can be accessed through Forms (see [[Python/Web Frameworks/Django/05 Accessing Request Data#|Accessing Request Data]])

# Static Files

Static files are configured from [[02 Setup and Settings.py#`Settings.py`|Settings.py]]

To serve Static Files, we need to create a `static` directory in our app directory

>[!tip] 
>Similarly to [[#Templates]] we need to create a sub directory in our `app_name/static` folder called `app_name`  
>Hence, the template should be at   
>`app_name/static/app_name/index.html`  
>We could just put static files in `static` but might cause naming conflict with multiple apps. This is just a name-spacing thing!
>Because of how the `AppDirectoriesFinder` static-file finder works, you can refer to this static file in Django as `app_name/style.css`

To load the static `css` into our Template, we can add the following at the top of our `html` file

```django
{% load static %}
<link rel="stylesheet" href="{% static 'polls/style.css' %}">
```

The `{% static %}` template tag generates the absolute URL of static files.

### Images
To load Images, we can create a subdirectory for images in `myapp/static/myapp`

We can add a file to this:
`myapp/static/myapp/images/background.png`

And reference this file in our stylesheet:
```css
body {
    background: white url("images/background.png") no-repeat;
}
```
