# Functional Views

### `request.POST` and `request.GET`
Not to be confused with the method of the request (`request.method`), which could be 'GET', 'POST', 'PUT', 'DELETE', etc. `request.GET` and `request.POST` are used to access the parameters of GET and POST requests, respectively.

path("<int:pk>/", views.DetailView.as_view(), name="detail"),

Body

Cookies

ETC...

# Class Based Views
[[08 Class Views]]

In a Django class-based view, you can access the request object via self.request inside any method of the class.

This works because Django's class-based views set self.request when the view is invoked, so it's available to all methods throughout the lifecycle of the view.
