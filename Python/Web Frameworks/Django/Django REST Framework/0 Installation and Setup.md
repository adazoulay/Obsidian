After [[Create python env]],  follow the [[02 Setup and Settings.py#Setup|Django Setup instructions]]

install the Django REST framework

```
pip install djangorestframework
```

After installation, add `rest_framework` to `installed_apps` in [[02 Setup and Settings.py#`Settings.py`|Settings.py]]
```python
INSTALLED_APPS = [
 ...
 'rest_framework'
      ]
```

