The return value from a view function is automatically converted into a response object for you
- If the return value is a string it’s converted into a response object with the string as response body, a `200 OK` status code and a _text/html_ mimetype
- If the return value is a dict or list, `jsonify()` is called to produce a response

The logic that Flask applies to converting return values into response objects is as follows
1. If a response object of the correct type is returned it’s directly returned from the view.
2. If it’s a string, a response object is created with that data and the default parameters.
3. If it’s an iterator or generator returning strings or bytes, it is treated as a streaming response.
4. If it’s a dict or list, a response object is created using `jsonify()`
5. If a tuple is returned the items in the tuple can provide extra information. Such tuples have to be in the form `(response, status)`, `(response, headers)`, or `(response, status, headers)`. The `status` value will override the status code and `headers` can be a list or dictionary of additional header values.
6. If none of that works, Flask will assume the return value is a valid WSGI application and convert that into a response object.

If you want to get hold of the resulting response object inside the view you can use the `make_response()` function:
```python
from flask import render_template

@app.errorhandler(404)
def not_found(error):
    return render_template('error.html'), 404
```

You just need to wrap the return expression with `make_response()` and get the response object to modify it, then return it:
```python
from flask import make_response

@app.errorhandler(404)
def not_found(error):
    resp = make_response(render_template('error.html'), 404)
    resp.headers['X-Something'] = 'A value'
    return resp
```

## APIs with JSON
If you return a `dict` or `list` from a view, it will be converted to a JSON response
```python
@app.route("/me")
def me_api():
    user = get_current_user()
    return {
        "username": user.username,
        "theme": user.theme,
        "image": url_for("user_image", filename=user.image),
    }

@app.route("/users")
def users_api():
    users = get_all_users()
    return [user.to_json() for user in users]
```
This is a shortcut to passing the data to the `jsonify()` function
