Django then looks at the URL patterns defined in the root URLconf file in the order they're defined, and uses the first one that matches the requested URL

These URL patterns map to view functions (see [[07 Views|Views]]), which are Python functions that take a `HttpRequest` object and return a `HttpResponse` object

To specify a path, we can import `path` from `django.urls` and place the desired path in the `urlpatterns` array

```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```

### `path()`

> [!info]
>Patterns don’t search GET and POST parameters, or the domain name
>For example, in a request to:
>	- `https://www.example.com/myapp/`
>	- `https://www.example.com/myapp/?page=3`,
> Will yield the same result. URLconf will look for `myapp/`. It goes with the first "hit"

The path function is passed **four** arguments

**Two** required:
- `route` : a string that contains  a URL Pattern. 
	- Starts at the first pattern in `urlpatterns` and makes its way down the list
	- Patterns don’t search GET and POST parameters, or the domain name
- `view`: function
	- When Django finds a matching pattern, it calls the specified view function with an `HttpRequest` object as the first argument and any “captured” values from the route as keyword arguments
	
**Two** optional:
- `kwargs`: dict
	- Arbitrary keyword arguments can be passed in a dictionary to the target
- `name`: string
	- Naming your URL lets you refer to it unambiguously from elsewhere in Django, especially from within templates
	- Allows you to make global changes to the URL patterns of your project while only touching a single file

## Path Arguments

To capture a value from the URL, use angle brackets
	`<variable_name>`
	
Captured values can optionally include a converter type
	`<type:variable_name>`
	
| Type  | Description                            |
|-------|----------------------------------------|
| str | Accepts any text without a slash        |
| int    | Accepts positive integers               |
| slug  | any slug string consisting of ASCII letters or numbers  |
| path   | Like string but also accepts slashes    |
| uuid   | Accepts UUID strings                    |

These captured parameters can then be used a a [[07 Views|view]] argument
	**Note:** Path arguments can also accept regex

`urls.py`
```python
urlpatterns = [
    path("", views.index, name="index"),
    path("<int:question_id>/", views.detail, name="detail"),
    path("<int:question_id>/results/", views.results, name="results"),
# ... 
```
Here, when someone requests a page from, for example:
	`/polls/34/results` 
Django will load the `mysite.urls` Python module because it’s pointed to by the `ROOT_URLCONF` setting in [[02 Setup and Settings.py#`Settings.py`|Settings.py]]
It finds the variable named `urlpatterns` and traverses the patterns in order until it finds a matching pattern

## Relative Paths

Our app can have it's own `urls.py` file
To add it to our app, we need to manually create a `urls.py` file

We can include our child paths relative to a root path using the `include()` method
```python
from django.contrib import admin
from django.urls import include, path

app_name = "polls" # Used for namespacing with multi apps 

urlpatterns = [
    path("polls/", include("polls.urls")),
    path("admin/", admin.site.urls),
]
```

### `Include()`
The `include()` function allows for referencing other URLconfs. Django chops off whatever part of the URL matched up to that point and sends the remaining string to the included URLconf for further processing

>[!warning]
 Always use `include()` when you include other URL patterns
 `admin.site.urls` is the only exception


## Paths in Class based Views
See ([[08 Class Views|class views]]) 

Class-based views are connected to URLs using the `as_view()` method, which returns a function that can be called when processing a request. The URLconf does not directly instantiate the class.

The most direct way to use generic views is to create them directly in your URLconf

```python
from django.urls import path
from django.views.generic import TemplateView

urlpatterns = [
	path("", views.IndexView.as_view(), name="index"),
    path("<int:pk>/", views.DetailView.as_view(), name="detail"),
    path("about/", views.TemplateView.as_view(template_name="about.html")),
]
```
In this case, `<int:pk>` captures a primary key value from the URL, which is passed to the `DetailView` and used to retrieve the `Question` instance with that primary key.

#### `as_view()`
Any arguments passed to `as_view()` will override attributes set on the class. In this example, we set `template_name` on the `TemplateView`

## Rerouting

Rerouting can be accomplished by returning an `HttpResponseRedirect` object in our [[07 Views|Views]]

`HttpResponseRedirect` takes a single argument: the URL to which the user will be redirected

```python
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question

# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
      #...
    else:
        selected_choice.votes += 1
        selected_choice.save()
        return HttpResponseRedirect(reverse("polls:results", args=(question.id,)))
```
In this example if the form is filled correctly, the user is redirected to the `results` page

#### `reverse()`
The  `reverse()` function helps avoid having to hardcode a URL in the view function

The first argument is matched to URL definition `name` parameter as specified in the `polls.urls` module (see [[Python/Web Frameworks/Django/03 Routing#`path()`|path()]])

This will match the URL for a [[07 Views|view]]  method, very similar to Flask's [[Python/Web Frameworks/Flask/03 Routing#URL Building|URL Building]] method `url_for()`

The output of this result call will return 
	`"/polls/3/results/"`
Where the `3` is the value of `question.id`



