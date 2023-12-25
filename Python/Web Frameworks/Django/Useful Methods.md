
## Convert Model to Dict

Can be done manually by building the dict
```python
from django.http import JsonResponse

import json
from products.models import Product

def api_home(request, *args, **kwargs):
    model_data = Product.objects.all().order_by("?").first()
    data = {}
    if model_data:
        data["id"] = model_data.id
        data["title"] = model_data.title
        data["content"] = model_data.content
        data["price"] = model_data.price
    return JsonResponse(data)

```

Or can be done automatically using `model_to_dict`
```python
from django.http import JsonResponse
from django.forms.models import model_to_dict

from products.models import Product

def api_home(request, *args, **kwargs):
    model_data = Product.objects.all().order_by("?").first()
    data = {}
    if model_data:
        data = model_to_dict(model_data, fields=['id','title'])
        # fields allows us to select desired model fields
    return JsonResponse(data)
```
