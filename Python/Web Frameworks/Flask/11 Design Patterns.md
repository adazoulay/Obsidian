## The application Factory 
Create the `flaskr` directory and add the `__init__.py` file
- The `__init__.py` serves double duty: it will contain the application factory, 
- Tells Python that the `flaskr` directory should be treated as a package
- When using `flask run` (See [[02 CLI#Running|Running]]), flask knows to go to the `__innit__.py` file as an entry point

`flaskr/__init__.py`
```python
import os
from flask import Flask


def create_app(test_config=None):
    # create and configure the app
    app = Flask(__name__, instance_relative_config=True)
    app.config.from_mapping(
        SECRET_KEY="dev",
        DATABASE=os.path.join(app.instance_path, "flaskr.sqlite"),
    )

    if test_config is None:
        # load the instance config, if it exists, when not testing
        app.config.from_pyfile("config.py", silent=True)
    else:
        # load the test config if passed in
        app.config.from_mapping(test_config)

    # ensure the instance folder exists
    try:
        os.makedirs(app.instance_path)
    except OSError:
        pass

    #! Example builtin route
    @app.route("/hello")
    def hello():
        return "Hello, World!"

    from . import db
    db.init_app(app)

    from . import auth
    app.register_blueprint(auth.bp)

    from . import blog
    app.register_blueprint(blog.bp)
    app.add_url_rule("/", endpoint="index")

    return app
```


## Global State

`db.py`
```python
import sqlite3
import click
from flask import current_app, g


def get_db():
    if "db" not in g:
        g.db = sqlite3.connect(current_app.config["DATABASE"], detect_types=sqlite3.PARSE_DECLTYPES)
        g.db.row_factory = sqlite3.Row
    return g.db


def close_db(e=None):
    db = g.pop("db", None)
    if db is not None:
        db.close()


def init_db():
    db = get_db()

    with current_app.open_resource("schema.sql") as f:
        db.executescript(f.read().decode("utf8"))
```
- In this example, the flask `g` object serves as a global namespace object for storing and sharing data during the life of a request context
	- Hence in our `__innit__.py` he can call the `init_db()` method to initialize our database which can be accessed later in the code by  using the `get_db()` method



## Blueprints
We can import blueprints (See [[Python/Web Frameworks/Flask/03 Routing#Blueprints|Blueprints]]) into our Application Factory to build relative paths from our main entry point

## Decorators
We can apply middleware similar to `Express.js` using **decorators**
```python
def login_required(view):
    @functools.wraps(view)
    def wrapped_view(**kwargs):
        if g.user is None:
            return redirect(url_for("auth.login"))

        return view(**kwargs)

    return wrapped_view

```
Here the `login_required(view)` method can be used as a decorator to make sure a user is authorized to make access/change some data
```python
@bp.route('/<int:id>/update', methods=('GET', 'POST'))
@login_required
def update(id):
    post = get_post(id)

    if request.method == 'POST':
        title = request.form['title']
	...
```

