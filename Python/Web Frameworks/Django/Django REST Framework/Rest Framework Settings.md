### DEFAULT_AUTHENTICATION_CLASSES

Accessible through [[02 Setup and Settings.py#`Settings.py`|Settings.py]]

A list or tuple of authentication classes, that determines the default set of authenticators used when accessing the request.user or request.auth properties.

Here are the defaults
```python
[
    'rest_framework.authentication.SessionAuthentication',
    'rest_framework.authentication.BasicAuthentication'
]
```

They can be modified like so:
```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [],
    "DEFAULT_PERMISSION_CLASSES": [],
}

```
These settings are analogous to: `permission_classes` and `authentication_classes`. See [[4 Session Authentication and Permissions#Adding Permissions]]