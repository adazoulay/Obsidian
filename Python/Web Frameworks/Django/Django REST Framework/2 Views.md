
# Functional

Functional views using DRF can handle different response types using the 
`@api_view` decorator. There are some similarities to Django [[07 Views|Views]]

```python
from django.forms.models import model_to_dict
from rest_framework.response import Response
from rest_framework.decorators import api_view

from products.models import Product


@api_view(["GET", "POST"])
def api_home(request, *args, **kwargs):
    instance = Product.objects.all().order_by("?").first()
    data = {}
    if instance:
        data = model_to_dict(instance, fields=["id", "title"])
    return Response(data)

```

In this example, the view will only be triggered on `GET` or `POST` requests removing the need to manually check `request.method` as seen in [[Python/Web Frameworks/Django/04 REST Methods#|Django Views, REST Methods]]


## Use with Serializers

### Getting Data From Serializers
Views can import [[1 Serializer|1 Serializer]] to easily convert ORM data to JSON to be returned

```python
from django.forms.models import model_to_dict
from rest_framework.response import Response
from rest_framework.decorators import api_view

from products.models import Product
from products.serializers import ProductSerializer


@api_view(["GET", "POST"])
def api_home(request, *args, **kwargs):
    instance = Product.objects.all().order_by("?").first()
    data = {}
    if instance:
        data = ProductSerializer(instance).data
    return Response(data)
```

Here, we pass the selected `instance` to our serializer and extract its data using the `.data` attribute

This data can then re returned using the DRF `Response` method 

### Sending Data To Serializers

Views are also able to send data to a [[1 Serializer]] 

```python
from django.forms.models import model_to_dict
from rest_framework.response import Response
from rest_framework.decorators import api_view

from products.models import Product
from products.serializers import ProductSerializer

@api_view(["POST"])
def api_home(request, *args, **kwargs):
    serializer = ProductSerializer(data=request.data)
    if serializer.is_valid(raise_exception=True):
        serializer.save()
        print(serializer.data)
        return Response(serializer.data)
    return Response({"invalid": "not good data"})
```

### Data Validation
We  check if the instance of the `serializer` is valid using `serializer.is_valid()`
This method checks weather the data is valid against the model
- We set `raise_exception=True` to give us descriptive error messages on invalid serializers
- Validation includes checking that required fields are included, that field values are of the expected data type, and that any additional field-specific constraints (such as a `max_length` on a `CharField`) are satisfied

If the data **is valid** we call `serializer.save()` which creates a new instance of our model and adds it to the database, we can then send back a `Respone` to the client

This data can then re returned using the DRF `Response` method 

# Class Based Views

Unlike Django [[08 Class Views|Class Views]] DRF class views inherit from the `rest_framework generics` class 

To build a View in DRF, we import the `generics` class and create a class inheriting from some generics

```python
from rest_framework import generics
from .models import Product
from .serializers import ProductSerializer

class ProductCreateApiView(generics.CreateAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer

    def perform_create(self, serializer):
        title = serializer.validated_data.get("title")
        content = serializer.validated_data.get("content")
        if content is None:
            serializer.validated_data["content"] = title
        serializer.save()

class ProductDetailAPIView(generics.RetrieveAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer

```

## Accessing Serializer Data from View
In `ProductCreateApiView`, we can access the serializer's data through the `validated_data` property
- `validated_data` is an attribute of the Serializer instance, which is a dictionary containing the deserialized values of the fields that were accepted as valid by the serializer
To see important Serializer methods, see [[#Common Serializer Methods]]

## Builtin attributes and methods

These classes come with prebuilt parameters we can set:
- `queryset`: This specifies the base query set that the view will use to filter results.
- `serializer_class`: This specifies the serializer class that will be used to convert model instances into JSON data for the HTTP response

As well as some overridable methods:
- `perform_create`:  Will only run on `CreateApiView` classes
	- This is a method on generic Django Rest Framework views that is called when creating a new instance.

Fore more details on common generic views, there attributes and methods see [[#Common Subclasses and  Attributes]] 

## Routing and HTTP Verbs
The routing in class based views is handled the same as in Django [[Python/Web Frameworks/Django/03 Routing#Relative Paths|Relative Paths]] and Views are triggered basted on the inherited generic view's builtin HTTP verb.
- Example: the `RetrieveAPIView` generic class view is set up to handle HTTP GET requests by default using the `retrieve()` method of the view

DRF's mapping from HTTP verbs to methods which can be overridden similarly to [[Python/Web Frameworks/Django/04 REST Methods#REST Methods in Class Based Views|Django Class View REST Methods]]
- GET: `list()` and `retrieve()`
- POST: `create()`
- PUT: `update()`
- PATCH: `partial_update()`
- DELETE: `destroy()`


# Common Subclasses and  Attributes

### ListAPIView:
Used for read-only endpoints to represent a collection of model instances. Responds to `GET` HTTP verb.

#### Fields:
- `queryset` (Mandatory): Defines the set of all items from which this view will pull data.
- `serializer_class` (Mandatory): Specifies the serializer to use for formatting output data.
- `filter_backends` (Optional): Used for defining custom filters.

#### Methods:
- `get_queryset(self)`: Can be overridden to customize the queryset at runtime.
- `list(self, request, *args, **kwargs)`: Represents the bulk of work done by the view, like querying the database and formatting output with the serializer. It calls `get_queryset()` to get the data to list.

### CreateAPIView:
Used for create-only endpoints to represent the creation of model instances. Responds to `POST` HTTP verb.

#### Fields:
- `queryset` (Mandatory): Defines the set of all items from which this view will pull data.
- `serializer_class` (Mandatory): Specifies the serializer to use for formatting input data.

#### Methods:
- `create(self, request, *args, **kwargs)`: Called when a new instance needs to be created. It calls `perform_create(serializer)` after the serializer's `is_valid()` has passed.
- `perform_create(self, serializer)`: Allows modifying how the instance save is managed.

### RetrieveAPIView:
Used for read-only endpoints to represent a single model instance. Responds to `GET` HTTP verb.

#### Fields:
- `queryset` (Mandatory): Defines the set of all items from which this view will pull data.
- `serializer_class` (Mandatory): Specifies the serializer to use for formatting output data.

#### Methods:
- `get_object(self)`: Used to retrieve the object the view displays.
- `retrieve(self, request, *args, **kwargs)`: Provides the data to respond with after retrieving the object. It calls `get_object()` to retrieve a specific instance.

### DestroyAPIView:
Used for delete-only endpoints to allow deletion of model instances. Responds to `DELETE` HTTP verb.

#### Fields:
- `queryset` (Mandatory): Defines the set of all items from which this view will pull data.
- `serializer_class` (Mandatory): Specifies the serializer to use for formatting output data.

#### Methods:
- `get_object(self)`: Used to retrieve the object the view will delete.
- `perform_destroy(self, instance)`: Called by `destroy()` to delete the object. Override this method to customize the delete process.
- `destroy(self, request, *args, **kwargs)`: Handles the deletion of the object and the response. It calls `get_object()` to get the instance to be deleted and `perform_destroy(instance)` to delete the instance.

### UpdateAPIView:
Used for update-only endpoints to represent updating a single model instance. Responds to `PUT` and `PATCH` HTTP verbs.

#### Fields:
- `queryset` (Mandatory): Defines the set of all items from which this view will pull data.
- `serializer_class` (Mandatory): Specifies the serializer to use for formatting input data.

#### Methods:
- `get_object(self)`: Used to retrieve the object the view will update.
- `update(self, request, *args, **kwargs)`: Handles the updating of the instance and the response. It calls `get_object()` to get the instance to be updated and `perform_update(serializer)` to update the instance.
- `perform_update(self, serializer)`: Allows modifying how the instance update is managed.

#### Methods:
- `get_object()`: Used to retrieve the object the view will update.
- `update()`: Handles the updating of the instance and the response. It calls `get_object()` to retrieve a specific instance and then calls `perform_update()` to save the changes to the instance.
- `perform_update(serializer)`: Allows modifying how the instance update is managed. Receives the validated serializer as an argument.

## Common Serializer Methods
- `data`: This attribute contains the serialized data (i.e., the output of the serialization process).
- `initial_data`: This attribute contains the input data that was used to initialize the Serializer (i.e., the input to the deserialization process).
- `errors`: After a serializer has been validated, this attribute contains any errors that were raised during validation. It's a dictionary where the keys are the field names and the values are lists of error messages.
- `is_valid(raise_exception=False)`: This method validates the `initial_data`. If the data is invalid, it will either return `False` or raise a `ValidationError` (if `raise_exception` is set to `True`).
- `save(**kwargs)`: This method saves the validated data into the model instance. You can also pass additional keyword arguments to be used in the save process.
- `create(validated_data)`: This method is used in the save process when creating a new instance. It should be overridden if you need any custom behavior when saving new objects.
- `update(instance, validated_data)`: This method is used in the save process when updating an existing instance. Like `create()`, you should override it if you need any custom save behavior for updates.