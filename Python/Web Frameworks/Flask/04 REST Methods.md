When accessing URLs (see [[Python/Web Frameworks/Flask/03 Routing|Routing]]) we can use different RESTFUL methods
By default,  `@app.route` answers to `GET` requests
```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return do_the_login()
    else:
        return show_the_login_form()
```
The example above keeps all methods for the route within one function, by [[Python/Web Frameworks/Flask/05 Accessing Request Data|Accessing Request Data]], which can be useful if each part uses some common data

We can also separate these requests into separate functions using builtin methods
	- `get()`, `post()`, `delete()`, `put()`, `patch()` 
```python
@app.get('/login')
def login_get():
    return show_the_login_form()

@app.post('/login')
def login_post():
    return do_the_login()
```
**Note:** If `GET`, support for `HEAD` methods automatically and `OPTIONS`

## Interacting with the DB

Interacting with the database depends on the type of DB being used in the project
```python
from flaskr.db import get_db

@bp.route("/create", methods=("GET", "POST"))
@login_required
def create():
    if request.method == "POST":
        title = request.form["title"]
        body = request.form["body"]
        error = None

        if not title:
            error = "Title is required."

        if error is not None:
            flash(error)
        else:
            db = get_db()
            db.execute("INSERT INTO post (title, body, author_id) VALUES (?, ?, ?)", (title, body, g.user["id"]))
            db.commit()
            return redirect(url_for("blog.index"))

    return render_template("blog/create.html")
```
In this example, the [[11 Design Patterns#Global State|Global State]] pattern is used so get access to the db using a `get_db()` method

From here, the [[Python/Web Frameworks/Django/03 Routing|route]] is able to interact with the db by executing databases