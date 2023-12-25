## Generic Class Based Views

```python
class ProductsMixinView(generics.GenericAPIView):
    
    def get(self, request, *args, **kwargs):
        #...

    def put(self, request, *args, **kwargs):
        #...

    def patch(self, request, *args, **kwargs):
        #...

    def delete(self, request, *args, **kwargs):
        #...

```

By adding `mixinxs` model views, such as `GenericAPIView`, we can now define a `queryset` and `seralizer`

```python
class ProductsMixinView(mixins.ListModelMixin, mixins.RetrieveModelMixin, mixins.CreateModelMixin, generics.GenericAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    lookup_field = "pk"

    # Handles `detail` and `list` views
    def get(self, request, *args, **kwargs):
        pk = kwargs.get("pk")
        if pk is not None:
            return self.retrieve(request, *args, **kwargs)
        return self.list(request, *args, **kwargs)

    # Handles `create` view
    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)

    #! You don't need This 
    #! can still implement to enhance slef.create() functionality
    
    def perform_create(self, serializer):
        title = serializer.validated_data.get("title")
        content = serializer.validated_data.get("content")
        if content is None:
            serializer.validated_data["content"] = "Missing content"
        serializer.save()
```
- In this example, `self.list`, `self.retrieve` comes from our respective inherited `mixin` methods, See [[2 Views#Routing and HTTP Verbs|HTTP Verbs]]
- In addition, methods like `perform_create` are still overridable to enhance the behavior of our views, See [[2 Views#Builtin attributes and methods|Builtin attributes and methods]]

>[! info]
>The reason methods like `perform_create` are available in our `mixins` is because [[2 Views#Class Based Views|generic API views]] extend both `GenericApiViews` and `...ModelMixin` views

## Mixins for Permissions
Mixins can also be used to handle [[4 Session Authentication and Permissions|Permissions]] 

```python
from .permissions import IsStaffEditorPermission
from rest_framework import permissions

class StaffEditorPermissionMixin:
    permission_classes = [permissions.IsAdminUser, IsStaffEditorPermission]

```

Then, in our [[2 Views#Class Based Views|Class Based Views]], we can import the mixin and incorporate it into our class
```python
class ProducListCreateApiView(
    generics.ListCreateAPIView,
    StaffEditorPermissionMixin,
):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
#...
```
We therefore don't need our `permission_classes` field anymore
