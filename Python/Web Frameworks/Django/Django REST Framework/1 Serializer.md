DRF (Django Rest Framework) uses serializers to convert Django's ORM querysets or objects to JSON format and vice-versa 

Serializers have their own fields just like Django [[08 Class Views#Common Subclasses|Class Views]] 

## Creating a Serializer

To create serializers, the convention is creating a `serializers.py` file in your app at the same level as `models.py`

`serializers.py` see [[#Reference Model]] 
```python
from rest_framework import serializers
from .models import Product

class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = [
            "title",
            "content",
            "price",
        ]
```
In this example, create a serializer by creating a class inheriting from `serializers.ModelSerializer` containing a sub class called `Meta`
- The sub-class needs to be called `Meta`

We set the `model` field to the desired model we are formatting and specify the fields we want to be returned in the response. 
- The fields array accepts, methods or decorated methods on our [[#Reference Model|Model]] 

Serializers can then be imported into [[2 Views]] 

## Serializer Methods and Properties
```python
class ProductSerializer(serializers.ModelSerializer):
    my_discount = serializers.SerializerMethodField(read_only=True)

    class Meta:
        model = Product
        fields = [...,"my_discount"]

	def get_my_discount(self, obj):
		print(obj.id)
		return obj.get_discount()
```
Here, we add a `SerializerMethodField` enabling us to rename the `get_discount()` property in our `Product` model
- `read_only=True` specifies that this field is only used when serializing the model instance [[2 Views#Sending Data To Serializers|Sending Data To Serializers]]

The `get_my_discount` method returns `get_discount()` as defined in the [[#Reference Model]]

>[!info]
>`self`  refers to the `ProductSerializer`
>`obj` refers to the instance of the model we are operating on

## Sending Data to the Serializer
[[2 Views]] are also able to send data to a serializer 

```python
class ProductSerializer(serializers.ModelSerializer):
    my_discount = serializers.SerializerMethodField(read_only=True)

    class Meta:
        model = Product
        fields = [
            "title",
            "content",
            "price",
            "sale_price",
            "my_discount",
        ]

    def get_my_discount(self, obj):
        if not hasattr(obj, "id"):
            return None
        return obj.get_discount()
```
Here, since our `get_discount()` method needs an instance of a `Product` to calculate the `get_discount()`, we check that the incoming `POST` request had valid data to create an instance of our `Product`
- We use the `hasattr()` method to check that `obj`, the current instance, exists and has an `id` associated with it

## Data Validation
Before creating an new instance of our new model, the serializer can perform [[2 Views#Data Validation|data validation]] through the matching view

## Custom Data Validation
We can validate serializer `fields` by creating methods to target said fields

The syntax is: `def validate_<fieldname>(self, value)`

In our example, this would look something like this:
```python
class ProductSerializer(serializers.ModelSerializer):
    # ...
    class Meta:
		 def validate_title(self, value):
		        qs = Product.objects.filter(title__iexact=value)
		        if qs.exclude():
		            raise serializers.ValidationError(f"{value} already a given name")
		        return value
```

We can add this as an existing serializer `Meta` class method or we can create create this method as a standalone method (without self) and use it in our serializer as a `serializers.CharField`
```python
from .validators import validate_title
class ProductSerializer(serializers.ModelSerializer):
    title = serializers.CharField(validators=[validate_title])
    class Meta:
        model = Product
        fields = [
            "title",
            # ...
```
>[!tip] 
>It's better to usually apply validators directly to the model
>`models.py`
>```python
>title = models.CharField(max_length=120, validators=[validate_title])
>```




## Common Serializers and their attributes

### Serializer:
It's the base serializer class from which all other serializers inherit. It allows complex data such as querysets and model instances to be converted to Python native data types that can then be easily rendered into JSON, XML or other content types. Responds to `is_valid()`, `save()` methods.

#### Fields:
- `required`: (Optional) A boolean that indicates whether the field must be present in the input. Default is `True`.
- `default`: (Optional) This provides the default value that should be used if no value is provided in the input.

#### Methods:
- `create()`: (Mandatory) Defines how fully validated data is turned into a model instance.
- `update()`: (Mandatory) Defines how fully validated data is used to update an existing model instance.
- `validate()`: (Optional) Can be overridden to provide custom field validation.

### ModelSerializer:
It's a shortcut for creating serializer classes with fields that correspond to the Model fields. The `ModelSerializer` class provides a set of default field mappings and a simple default `.create()` and `.update()` implementations.

#### Fields:
- `model`: (Mandatory) Defines the model to create the Serializer off of.
- `fields`: (Mandatory) Defines which fields on the Model should be included in the serialized output.

#### Methods:
- Inherits all methods from the `Serializer` class.

### ListSerializer:
A serializer class that provides a way of serializing multiple objects in the same request.

#### Fields:
- `child`: (Mandatory) This is a serializer instance that will be used for each item in the list.

#### Methods:
- `create()`: (Optional) Overridden to handle creating multiple objects.
- `update()`: (Optional) Overridden to handle updating multiple objects.

### StringRelatedField, SlugRelatedField, PrimaryKeyRelatedField, etc.:

These are related fields that allow you to represent a model's relations in various ways, such as using a string representation, using a slug, or using a primary key.
#### Fields:

- `queryset`: (Optional) If set, this queryset will be used to validate user input.

#### Methods:
- `to_representation()`: (Optional) Can be overridden to change how the field is represented in serialized output.
- `to_internal_value()`: (Optional) Can be overridden to change how user input is validated and converted into Python objects.


## Reference Model
`models.py`
```python
from django.db import models

class Product(models.Model):
    title = models.CharField(max_length=120)
    content = models.TextField(blank=True, null=True)
    price = models.DecimalField(max_digits=15, decimal_places=2, default=99.99)

    @property
    def sale_price(self):
        return "%.2f" % (float(self.price) * 0.8)

    def get_discount(self):
        return "122"

```
