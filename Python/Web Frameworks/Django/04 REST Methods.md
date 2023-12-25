To filter different types of requests, ie `GET`, `POST`, `DELETE`, etc.., we can use a [[07 Views|View]]'s `request` parameter

We can access a request's HTTP method through `request.method` and handle the request based on the type of request 
```python
from django.http import HttpResponse

def my_view_get(request):
    if request.method == 'GET':
        return HttpResponse('GET request received!')
    else:
        return HttpResponse('Invalid request')
```


### `request.POST` and `request.GET`
Not to be confused with the method of the request (`request.method`), which could be 'GET', 'POST', 'PUT', 'DELETE', etc. `request.GET` and `request.POST` are used to access the parameters of GET and POST requests, respectively.
```python
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST["choice"])
	#...
```

Django's `httpRequest` object has:

- `request.POST` a dictionary-like object that lets you access submitted data by key name
	- used when a form is submitted and the data needs to be handled in the view
	- In this case, `request.POST['choice']` returns the ID of the selected choice, as a string
	
- `Request.GET`  a dictionary-like object that lets you access submitted data by key name
	- Allows you to access GET parameters (i.e., values in the query string of a URL)
	- It's usually used when you're trying to read parameters from a URL.

## REST Methods in Class Based Views

Most [[08 Class Views|Class Views]] come with some sensible default behaviors, so these are available to be overridden

In the Django web framework, a view's HTTP method handling is done through methods on the view itself. For example, if a request is a GET, Django would call the `get()` method on the view

`get(self, request, *args, **kwargs)`: This method is called by Django to handle a GET request.

`post(self, request, *args, **kwargs)`: This method is called by Django to handle a POST request.
    
`put(self, request, *args, **kwargs)`: This method is called by Django to handle a PUT request.
    
`patch(self, request, *args, **kwargs)`: This method is called by Django to handle a PATCH request.
    
`delete(self, request, *args, **kwargs)`: This method is called by Django to handle a DELETE request.
    
`head(self, request, *args, **kwargs)`: This method is called by Django to handle a HEAD request.
    
`options(self, request, *args, **kwargs)`: This method is called by Django to handle an OPTIONS request.
    
`trace(self, request, *args, **kwargs)`: This method is called by Django to handle a TRACE request.
