# Viewsets
```python
class ProductViewSet(viewsets.ModelViewSet):
    """
    get -> list -> Queryset
    get -> retrieve -> Product Instance Detail View
    put -> create -> New Instance
    put -> update
    patch -> Partial Update
    delete -> destroy
    """
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    lookup_field = "pk"
```
- Implementing `ModelViewSet` yields support to all crud operations 
- `lookup_fied` uses the serializer to get a `Product` instance with primary key 1
	- `http://127.0.0.1:8000/api/v2/products-abc/1/`

We can also use [[3 Mixins|Mixins]] by using the `viewsets.GenericViewSet`
```python
class ProductGenericViewset(
    viewsets.GenericViewSet,
    mixins.RetrieveModelMixin,
    mixins.CreateModelMixin,
):
    """
    get -> list -> Queryset
    get -> retrieve -> Product Instance Detail View
    """
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    lookup_field = "pk"

```
- In this case only the `get`: `list` and `retrieve` operations are supported

>[!tip]
>Since it can be confusing to remember what `ViewSet` supports what operations, we can destructure it like so:
>```python
>	product_list_view = ProductGenericViewset.as_view({"get": "list"})
>	product_detail_view = ProductGenericViewset.as_view({"get": "retrieve"})
>```
>Where the verbs used in the dictionary `list`, `retrieve` match those in [[2 Views#Routing and HTTP Verbs|Routing and HTTP Verbs]]
>We can then map these variables to classic urlConf routes [[Python/Web Frameworks/Django/03 Routing#`path()`|Routing]]


# Routers
`routers.py`
```python
from rest_framework.routers import DefaultRouter
from products.viewsets import ProductViewSet

router = DefaultRouter()
router.register("products-abc", ProductViewSet, basename="products")

urlpatterns = router.urls
print(urlpatterns)
```
- Routers don't provide the granular precision [[Python/Web Frameworks/Django/03 Routing#|Routing]]

To link this router to our routes, we can simply `include` it
`urls.py`
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", include("api.urls")),
    path("api/products/", include("products.urls")),
    path("api/v2/", include("cfehome.routers")),
]
```