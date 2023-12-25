## Class Based View

Class views, or generic views allow fore more separation of concern as compared to functional [[07 Views|Views]]

All class based views inherit from the `View` class which handles linking the view into the URLs, HTTP method dispatching and other common features
-  `RedirectView` provides a HTTP redirect, and `TemplateView` extends the base class to make it also render a template

Class-based views are organized around models, providing pre-defined methods for handling typical HTTP methods like GET, POST, PUT, DELETE, etc. Developers create views by subclassing `View` or one of its many subclasses that implement a particular type of functionality.

```python
from django.views.generic import View

class NewPostView(View):
    def render(self, request):
        return render(request, 'new_post.html', {'form': self.form})

    def post(self, request):
        self.form = PostForm(request.POST)
        if self.form.is_valid():
            self.form.save()
            return redirect('post_list')
        return self.render(request)

    def get(self, request):
        self.form = PostForm()
        return self.render(request)
```

## Generic Class-Based View

```python
class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by("-pub_date")[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = "polls/detail.html"

```
- By default, the `DetailView` and `ListView` generic views uses a template called `<app name>/<model name>_detail.html`. 
- In our case, it would use the template `polls/question_detail.html`

Here we see two types of Class views, a `TemplateView` and a `ListView` 
- `template_name`, `contex_object_name` and `model` are all examples of subclass attributes
- `get_queryset()` is an example of a builtin overrideable method
See ([[#Common Subclass Attributes]])

### Common Subclasses
- `TemplateView`: Used for displaying a template. It has a `get` method which renders a given template with any extra parameters provided.
- `ListView`: Used for displaying a list of objects. It automatically fetches the query-set for a given model and passes it to the template.
- `DetailView`: Used for displaying a single object. It fetches an object based on the provided primary key and passes it to the template.
- `FormView`: Used for handling form submissions. It handles displaying a blank form, validating form submissions, and performing an action (like saving data) when the form is valid.
- `CreateView`, `UpdateView`, `DeleteView`: Used for creating, updating, and deleting an object respectively. They handle fetching the object, displaying the form, and saving or deleting the object.

## Use in URLConf
(See [[Python/Web Frameworks/Django/03 Routing#Paths in Class based views|Paths in Class based views]])
The most direct way to use generic views is to create them directly in your URLconf
```python
from django.urls import path
from django.views.generic import TemplateView

urlpatterns = [
    path("about/", TemplateView.as_view(template_name="about.html")),
]
```

## Mixins 
Class-based views also support mix-ins, which allow you to add functionality to a view by deriving from multiple classes. Common mix-ins include `LoginRequiredMixin` (which restricts a view to logged-in users) and `FormMixin` (which adds form handling methods).

## HTTP Verbs:
Most Class based views come with prebuilt HTTP Verbs that can be overridden 
See [[Python/Web Frameworks/Django/04 REST Methods#REST Methods in Class Based Views|REST Methods in Class Based Views]]

# Common Subclass Attributes

## **TemplateView**:
###### Methods
- `get_template_names()`: Optional. The default implementation returns `self.template_name`.
- `get_context_data()`: Optional. The default implementation includes the page object (i.e., the list of objects) and the paginator (if pagination is enabled).
###### Attributes
- `template_name`: Specifies the name of the template to use.

## **ListView**:
###### Methods
- `get_queryset()`: Optional. The default implementation returns `self.queryset` if provided, otherwise it uses the `model` attribute to fetch all objects.
- `get_context_data()`: Optional. The default implementation includes the page object (i.e., the list of objects) and the paginator (if pagination is enabled).
###### Attributes
- `model`: Specifies the model to retrieve objects.
- `context_object_name`: Specifies the context name for the list of objects in the template.
- `template_name`: Specifies the name of the template to use.
- `paginate_by`: Specifies the number of objects per page when pagination is enabled.

## **DetailView**:
###### Methods
- `get_object()`: Optional. The default implementation requires that `self.queryset` is provided and that the URLconf includes a `pk` or `slug`.
###### Attributes
- `model`: Specifies the model to retrieve objects.
- `context_object_name`: Specifies the context name for the object in the template.
- `template_name`: Specifies the name of the template to use.
- `pk_url_kwarg`: Specifies the name of the URL parameter that contains the primary key.

## **CreateView** and **UpdateView**:
###### Methods
`get_success_url()`: Optional. The default implementation returns `self.success_url`.
###### Attributes
- `model`: Specifies the model to create or update objects.
- `form_class`: Specifies the form class to use for object creation or updating.
- `template_name`: Specifies the name of the template to use.
- `success_url`: Specifies the URL to redirect to after successful form submission.

## **DeleteView**:
###### Methods
`get_success_url()`: Mandatory. You must either set `success_url` or override this method to return a valid URL to redirect to after deletion.
###### Attributes
- `model`: Specifies the model to delete objects.
- `context_object_name`: Specifies the context name for the object to delete.
- `template_name`: Specifies the name of the template to use.
- `success_url`: Specifies the URL to redirect to after successful deletion.

## **RedirectView**:
###### Methods
###### Attributes
- `url`: Specifies the URL to redirect to.
- `permanent`: Specifies whether the redirection is permanent (HTTP 301) or temporary (HTTP 302).


# Visual Representation
```
View (Django's base view class)
|
|--- TemplateView (Renders a given template with a context)
|    |
|    |--- ListView (Renders a list of objects, given a model)
|    |
|    |--- DetailView (Renders a "detail" view of a single object)
|
|--- FormView (Renders a form on GET and handles its submission on POST)
|    |
|    |--- CreateView (A view for creating new objects)
|    |
|    |--- UpdateView (A view for updating an object, with a response rendered by template)
|    |
|    |--- DeleteView (A view for deleting an object)
|    
|--- RedirectView (Provides a redirect on any GET request)
|
|--- View (Other user defined views)
```