  # Create a minimal Django app

## Running 
`python manage.py runserver`

* Starts the development server
* Support for hot reloading builtin

## Creating the Environment/Project

Make a new directory called `myproject` and `cd` into it 
Then run:

`django-admin startproject mysite .`

- Creates `manage.py` : the Django command-line administrative utility for the project
	- You run administrative commands for the project using `python manage.py <command> [options]`
- A subfolder named `mysite`, which contains the following files:
	- `__innit__.py` : empty file, tells Python this folder is a package
	- `asgi.py` : an entry point for ASGI-compatible web servers to serve your project (leave as is)
	- `settings.py` : contains settings for Django project, which you modify (See [[02 Setup and Settings.py#`Settings.py`|Setup]])
	- `urls.py`: contains a table of contents for the Django project, which you also modify
	- `wsgi.py`: an entry point for WSGI-compatible web servers to serve your project (leave as is)

## Creating an App

To create an App, run:

`python manage.py startapp myapp`

>[!tip]
Make sure you are in the same directory as `manage.py` to enable top-level module import

- Creates a `myapp` directory laid out like so:
```
myapp/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```
* This directory structure houses the `myapp` application

## Create an empty dev database

	`python manage.py migrate`

* Creates a SQLite database in `db.sqlite3` file by default
* migrate command looks at the [[02 Setup and Settings.py#Other Settings|INSTALLED_APPS]] settings and creates any necessary database tables according to the database settings in `settings.py`

## Migrating a database

	`python manage.py makemigrations myapp`

- Stages the migration, ie changes to models/DB
- This command takes Django that models have been modified and you want the changes to be stored as a migration
- Migrations are how Django stores changes to your models (and thus your database schema) - they’re files on disk

Then to apply the changes:

`python manage.py migrate`

## Django API

To access the Django API, use

	`python manage.py shell`

See [[11 Django API|Django API]]

## Creating a Django Admin

To create a Django admin, use

	`python manage.py createsuperuser`

See [[10 Django Admin|Django Admin]]

