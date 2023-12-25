Request data can be sent through the request object or through url parameters

# URL Parameters
We can access url parameters in our routing functions: (See [[Python/Web Frameworks/Flask/03 Routing#Variable Rules|Variable Rules]])
```python
@app.route('/user/<username>')
def show_user_profile(username):
	print(username)
	...
```
The data here can be passed through [[Jinja#Forms]] 


# The Request Object

For web-apps we need to react to the data a client sends to the server
This is done through the `request` object

To access the `request` object, you first need to import it 
```python
from flask import request
```

## `method` and `form` attributes
The current request method is available by using the `method` attribute (example: [[Python/Web Frameworks/Flask/04 REST Methods|REST Methods]])
To access form data, (the data transmitted in a `POST` or `PUT` request), we can use the `form` attribute
Here is a full example of both:
```python
@app.route('/login', methods=['POST', 'GET'])
def login():
    error = None
    if request.method == 'POST':
        if valid_login(request.form['username'],
                       request.form['password']):
            return log_the_user_in(request.form['username'])
        else:
            error = 'Invalid username/password'
    # the code below is executed if the request method
    # was GET or the credentials were invalid
    return render_template('login.html', error=error)
```
>_**Note:** `request.form` is analogous to Express.js `req.body`

In order to dispatch a method (ie `POST`, `GET`, ...) to the route to populate `request.form` data, [[Jinja#Forms|Jinja Forms]] has builtin send support

## `args` atribute
To access parameters submitted in the URL (`?key=value`) you can use the `args` attribute:
```python
searchword = request.args.get('key', '')
```
> **Note:** `request.args` is analogous to Express.js `req.query`

## File Uploads
<span style="color: red">Note: </span> In order to handle uploaded files you need to set the `enctype="multipart/form-data"`  attribute in the HTML form  

Uploaded files are stored in memory or at a temporary location on the filesystem
You can access those files by looking at the `files` attribute on the request object
Each uploaded file is stored in the **dictionary**

It behaves just like a standard Python `file` object, but it also has a `save()` method that allows you to store that file on the filesystem of the server
```python
from flask import request

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/uploaded_file.txt')
    ...
```


## Cookies
To access cookies, you can use the `cookies` attribute
To set cookies you can use the `set_cookie` method of response objects
> If you want to use sessions, don't use cookies but instead use [[08 Sessions|Sessions]]

**Reading Cookies**: 
```python
from flask import request

@app.route('/')
def index():
    username = request.cookies.get('username')
    # use cookies.get(key) instead of cookies[key] to not get a
    # KeyError if the cookie is missing.
```

**Storing Cookies**: First need to create the [[07 Response|Response]]
```python
from flask import make_response

@app.route('/')
def index():
    resp = make_response(render_template(...))
    resp.set_cookie('username', 'the username')
    return resp
```


### Other important properties
Some of the important properties and methods of Flask's request object include:
- `request.method`: The HTTP method used in the request.
- `request.args`: A dictionary-like object containing all provided HTTP GET parameters.
- `request.form`: A dictionary-like object containing all HTTP POST parameters.
	- See [[Jinja#Forms]]
- `request.files`: A dictionary-like object containing all uploaded files.
- `request.json`: If the mimetype is application/json, this contains the parsed JSON data.
	- See [[07 Response#APIs with JSON|APIs with JSON]]
- `request.headers`: A dictionary-like object containing the request headers.
- `request.cookies`: A dictionary-like object containing all cookies sent by the client.
- `request.url`: The full URL requested by the client.