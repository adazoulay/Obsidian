To route to different pages, we use the `app.route()` decorator to bind a function to a URL
```python
@app.route('/')
def index():
    return 'Index Page'

@app.route('/hello')
def hello():
    return 'Hello, World'
```

## Variable Rules
We can add variable sections to a URL by marking sections with
	`<variable_name>`
The functions then receive `<variable_name>` as a keyword argument

We can also use a **converter** to specify the type of the argument
	`<converter:variable_name>`

**Note** If the type of the parameter does not match the one in path, the route won't be executed

| Type  | Description                            |
|-------|----------------------------------------|
| string | Accepts any text without a slash        |
| int    | Accepts positive integers               |
| float  | Accepts positive floating point values  |
| path   | Like string but also accepts slashes    |
| uuid   | Accepts UUID strings                    |

Example:
```python
from markupsafe import escape

@app.route('/user/<username>')
def show_user_profile(username):
    # show the user profile for that user
    return f'User {escape(username)}'

@app.route('/post/<int:post_id>')
def show_post(post_id):
    # show the post with the given id, the id is an integer
    return f'Post {post_id}'

@app.route('/path/<path:subpath>')
def show_subpath(subpath):
    # show the subpath after /path/
    return f'Subpath {escape(subpath)}'
```
This data can be passed through [[Jinja#Forms|Jinja Forms]] using [[Python/Web Frameworks/Flask/03 Routing#URL Building|URL Building]]

## Unique URLs / Redirection Behavior
```python
@app.route('/projects/')
def projects():
    return 'The project page'

@app.route('/about')
def about():
    return 'The about page'
```
- The first endpoint has a trailing '/'
	- If you access the URL without a trailing slash (`/projects`), Flask redirects you to the canonical URL with the trailing slash (`/projects/`)
- The second endpoint does not have a trailing '/'
	- Accessing the URL with a trailing slash (`/about/`) produces a 404 “Not Found” error

## URL Building
To build a URL to a specific function, we use the `url_for()` function
- Accepts the name of the function as its first argument and any number of keyword arguments, each corresponding to a variable part of the URL rule
##### **Why use it?**
- More descriptive than hard coding
- Can change URLs in one go 
- URL building handles escaping of special characters transparently
- The generated paths are always absolute
- If your application is placed outside the URL root, for example, in `/myapplication` instead of '/',  `url_for()` properly handles that for you

**Example**: We use the `test_request_context()` method to test `url_for()`
- `test_request_context()` tells Flask as though it's handling a request 
```python
from flask import url_for

@app.route('/')
def index():
    return 'index'

@app.route('/login')
def login():
    return 'login'

@app.route('/user/<username>')
def profile(username):
    return f'{username}\'s profile'

with app.test_request_context():
    print(url_for('index'))
    print(url_for('login'))
    print(url_for('login', next='/'))
    print(url_for('profile', username='John Doe'))
```
The output here is 
```
/
/login
/login?next=/
/user/John%20Doe
```

## Redirecting and Errors
To redirect a user to another endpoint, use the `redirect()` function
To abort a request early with an error code, use the `abort()` function
```python
from flask import abort, redirect, url_for

@app.route('/')
def index():
    return redirect(url_for('login'))

@app.route('/login')
def login():
    abort(401)
    this_is_never_executed()
```

By default a black and white error page is shown for each error code. If you want to customize the error page, you can use the `errorhandler()` decorator:
```python
from flask import render_template

@app.errorhandler(404)
def page_not_found(error):
    return render_template('page_not_found.html'), 404
```


## Blueprints
To achieve relative paths similar to Express.js `app.use(relative_path)`, we can use Blueprints

`Blueprints` group a set of related routes, and then registering that blueprint with a given URL prefix when you're ready to attach it to the application

```python
from flask import Flask, Blueprint

bp = Blueprint('my_blueprint', __name__)

@bp.route('/path1')
def function1():
    return "You've reached function1"

@bp.route('/path2')
def function2():
    return "You've reached function2"

app = Flask(__name__)
app.register_blueprint(bp, url_prefix='/prefix')

```

In this example, the routes defined in `my_blueprint` are '/path1' and '/path2', 
However, when `my_blueprint` is registered to the application with the URL prefix '/prefix', they become '/prefix/path1' and '/prefix/path2' respectively