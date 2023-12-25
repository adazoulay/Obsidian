To create views, we can add them to our app's `views.py` file (see [[01 CLI#Creating an App|CLI]] )

In Django, a "view" refers to a Python function or class that takes a web request and returns a web response

Each view is responsible for doing one of two things: 
- returning an `HttpResponse` object containing the content for the requested page, or raising an exception such as `Http404`

> The `HttpResponse` object, can contain: HTML content, a `JsonResponse` object, containing JSON content, a redirect to another view [[Python/Web Frameworks/Django/03 Routing#Rerouting|Rerouting]], a 404 [[#Raising a 404 error]]

Simple example `views.py`
```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")

```

These views can take in an argument: `views.py`
```python
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)
```

### Linking to Paths

These views can then be triggered and pass arguments through urlPatterns:  See ([[Python/Web Frameworks/Django/03 Routing#Path Arguments|Routing]])

When a pattern in a requested URL matches, the matching View is called, for example, a call to `detail` at url: `/polls/34/`would look like so:
```python
detail(request=<HttpRequest object>, question_id=34)
```
The `question_id=34` part comes from `<int:question_id>`.
Using angle brackets “captures” part of the URL and sends it as a keyword argument to the view function.


## Interacting with Models

To access and modify/return data from our [[06 Models|Models]], we can import them into `views.py`
```python
from django.http import HttpResponse
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    output = ", ".join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
```
From here, we can perform operations to extract/mutate data in our models and return an `HttpResponse`

(See [[06 Models#Crud Operations|Crud Operations]])

## Rendering Templates

We can return [[09 Templates and Static Files]] in an `HttpResponse` object like so:
```python
from django.http import HttpResponse
from django.template import loader
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    template = loader.get_template("polls/index.html")
    context = {
        "latest_question_list": latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```
That code loads the template called `polls/index.html` and passes it a context
> Context is a dictionary mapping template variable names to Python objects, in this case, the `latest_question_list` matches the variable name in the template

#### A shortcut: `render()`
Django provides a shortcut to loading a [[09 Templates and Static Files|Template]], filling a context and returning an `HttpResponse` object with the result of the rendered template
```python
from django.shortcuts import render
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    context = {"latest_question_list": latest_question_list}
    return render(request, "polls/index.html", context)
```
- We no longer need to import `loader` and `HttpResponse` 
- `render()` takes the request object as its first argument, a template name as its second argument and a dictionary as its optional third argument
- Returns an `HttpResponse` object of the given template rendered with the given context

## Raising a 404 error
```python
from django.http import Http404
from django.shortcuts import render
from .models import Question

def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, "polls/detail.html", {"question": question})
```
Here, The view **raises** the Http404 `exception` if a question with the requested ID doesn’t exist

#### A Shortcut: `get_object_or_404()`
It’s a very common idiom to use `get()` and raise `Http404` if the object doesn’t exist. Django provides a shortcut
```python
from django.shortcuts import get_object_or_404, render
from .models import Question

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/detail.html", {"question": question})
```


## Class Views
See [[08 Class Views|Class Views]]

