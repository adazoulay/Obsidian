The rest framework comes with builtin class [[2 Views|Views]] and [[3 Mixins|Mixins]] attributes to handle authentication and permissions

# Validating Permissions

`permission_classes`: This is a list of classes that the Django REST framework will use to authorize the request after authentication. 
Authorization is about determining what the authenticated client has the rights to do

```python
from rest_framework import generics, mixins, permissions


class ProducListCreateApiView(generics.ListCreateAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [permissions.IsAdminUser, IsStaffEditorPermission]

    def perform_create(self, serializer):
```
- In this example, sending a request to this endpoint will return a `403` if the user is not authenticated
- We could replace `.IsAuthenticated` with `.IsAuthenticatedOrReadOnly` to allow `GET` requests to get through 
>[!tip]
>The permissions in the `permission_classes` field get matched in order. 
>So order is important 

## Adding Permissions

`authentication_classes`: This is a list of classes that the Django REST framework will use to authenticate the client making the request. 

Authentication is about determining the identity of the client. It involves checking credentials like username and password, tokens, or any other mechanism that confirms the client is who they claim to be

```python
class ProducListCreateApiView(generics.ListCreateAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    authentication_classes = [authentication.SessionAuthentication]
```

Both `permission_classes` and `authentication_classes` default behavior can be modified through [[Rest Framework Settings#DEFAULT_AUTHENTICATION_CLASSES|Rest Framework Settings]]

# Custom Permissions

To create custom permissions, we can create a `permissions.py` file in our app directory

We can the access the user data from the `request.user` parameter in builtin overridable methods:
```python
class IsStaffEditorPermission(permissions.DjangoModelPermissions):
    perms_map = {
        "GET": ["%(app_label)s.view%(model_name)s"],
        "OPTIONS": [],
        "HEAD": [],
        "POST": ["%(app_label)s.add_%(model_name)s"],
        "PUT": ["%(app_label)s.change_%(model_name)s"],
        "PATCH": ["%(app_label)s.change_%(model_name)s"],
        "DELETE": ["%(app_label)s.delete_%(model_name)s"],
    }

    def has_permission(self, request, view):
        if not request.user.is_staff:
            return False
        return super().has_permission(request, view)

    def has_object_permission(self, request, view, obj):
        return True

```


### Adding Permissions to Views

To add these permissions to our views, we can import them and add them to our `permission_classes` array 

```python
from .permissions import IsStaffEditorPermission

class ProducListCreateApiView(generics.ListCreateAPIView):
	#...
    permission_classes = [IsStaffEditorPermission]
```


# Token Authentication
To add token authentication, we need to add a `"rest_framework.authtoken"` entry to our [[02 Setup and Settings.py#`Settings.py|Settings.py]] `INSTALLED_APPS` array and run a migration [[01 CLI#Migrating a database|Migrating a database]]

We can then create an endpoint to generate auth tokens:
```python
from rest_framework.authtoken.views import obtain_auth_token
urlpatterns = [
    path("", views.api_home),
    path("auth/", obtain_auth_token),
]
```

We then need to add `authentication.TokenAuthentication` to our `authentication_classes` field in our generic view
```python
class ProducListCreateApiView(generics.ListCreateAPIView):
    #...
    authentication_classes = [
        authentication.SessionAuthentication,
        authentication.TokenAuthentication,
    ]
```
Now, if we make a request to this endpoint without an auth token, our request is rejected

If we want to make a request to get our list, the flow now looks something like this:
```python
auth_endpoint = "http://localhost:8000/api/auth/"
username = input("What is your username?\n")
password = getpass("What is your password?\n")

auth_response = requests.post(auth_endpoint, json={"username": username, "password": password})
print(auth_response.json())

if auth_response.status_code == 200:
    token = auth_response.json()["token"]
    headers = {"Authorization": f"Bearer {token}"}
    endpoint = "http://localhost:8000/api/products/"

    get_response = requests.get(endpoint, headers=headers)
```
- We use the `get_pass` method to encrypt our password from the CLI

#### Overriding Auth Token
To do so, create an `authentication.py` file and create a new class that inherits from `BaseAuthentication`
```python
from rest_framework.authentication import TokenAuthentication as BaseTokenAuth

class TokenAutentication(BaseTokenAuth):
    keyword = "Bearer"

```
We can then add it to our Class `authentication_class` field allowing us to refer to the token like so:
```python
headers = {"Authorization": f"Bearer {token}"}
```

## Mixins for Permissions
[[3 Mixins|Mixins]] can also be used to handle Permissions

