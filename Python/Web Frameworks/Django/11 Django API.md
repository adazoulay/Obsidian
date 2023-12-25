To invoke the Python shell, use this command

`python manage.py shell`

We can then use the the [database API](https://docs.djangoproject.com/en/4.2/topics/db/queries/) in an interactive python shell

Example:
```shell
 >>> from polls.models import Choice, Question 
 >>> from django.utils import timezone
 >>> q = Question(question_text="What's new?", pub_date=timezone.now())
 >>> q.save()
 >>> q.id
 1
 >>> q.question_text
 "What's new?"
 >>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=datetime.timezone.utc)
>>> q.question_text = "What's up?"
>>> q.save()
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```

In this example, the output to the last query is:
	`<QuerySet [<Question: Question object (1)>]>`

In order to get more readable step, we can add a `__str()__` method to our [[06 Models]]

```python
from django.db import models

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```


### Commands

#### Creating an entry:
```python
from myapp.models import Product
Product.objects.create(title="Hello",...,...)
```

### Get all Model  Entries
```python
Product.objects.all()
```