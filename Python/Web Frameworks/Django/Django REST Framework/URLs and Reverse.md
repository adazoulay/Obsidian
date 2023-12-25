The DRF `reverese()` method is slightly different to the Django builtin [[Python/Web Frameworks/Django/03 Routing#`reverse()`|reverse()]] method

```python
from rest_framework.reverse import reverse
```

We can use it in a [[1 Serializer|Serializer]] to return the request Url as a field
```python
from rest_framework import serializers
from rest_framework import reverse
from .models import Product

class ProductSerializer(serializers.ModelSerializer):
    my_discount = serializers.SerializerMethodField(read_only=True)
    url = serializers.SerializerMethodField(read_only=True)

    class Meta:
        model = Product
        fields = [
            "url",
	         #...
        ]

    def get_url(self, obj):
        request = self.context.get("request")
        if request is None:
            return None
        return reverse("product-detail",
            kwargs={"pk": obj.pk},
            request=request)
```
- The firs argument matches the name of the route view, in this case the `product-detail` view. See [[#Ref URLs.py]]
- The `pk` `kwarg` field matches the request's `pk` arg

>[! info]
> Here we access the request through `self.context.get("request")`
> If `self.request` was used, it would imply that the `request` object is a direct attribute of the Serializer, which it is not 

### Alternative to `reverse()`
We can instead also use the `HyperlinkedIdentityField` field which accomplishes the same thing 
```python
# ...
class ProductSerializer(serializers.ModelSerializer):
    url = serializers.HyperlinkedIdentityField(
        view_name="product-detail",
        lookup_field="pk",
    )

    class Meta:
        model = Product
        fields = [
            "url",
            # ..
```


# Ref Urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path(
        "",
        views.ProducListCreateApiView.as_view(),
        name="product-list",
    ),
    path(
        "<int:pk>/update/",
        views.ProductUpdateAPIView.as_view(),
        name="product",
    ),
    path(
        "<int:pk>/delete/",
        views.ProductDeleteView.as_view(),
        name="product",
    ),
    path(
        "<int:pk>/",
        views.ProductDetailAPIView.as_view(),
        name="product-detail",
    ),
]
```