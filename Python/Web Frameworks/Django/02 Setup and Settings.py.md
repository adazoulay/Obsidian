## Setup

1. Use the CLI to [[01 CLI#Creating the Environment/Project|Create the Environment/Project]] then use the CLI to [[01 CLI#Create a minimal Django app|Create a minimal Django app]] 
2. Run your initial migrations of the built-in user model using [[01 CLI#Create an empty dev database|this command]]
3. Create a `urls.py` file in our app directory and begin [[Python/Web Frameworks/Django/03 Routing#Relative Paths|Relative Paths]] 

## `Settings.py`
Python module with module-level variables representing Django settings created using [[01 CLI#Creating the Environment/Project|CLI]]

### Database Settings
By default, the configuration uses SQLite
If you want to switch to a different DB, change the following keys
- `DATABASES` `'default'` item to match your database connection settings
- `NAME` the name of the database
- If not using `SQLite`, additional settings such as `USER`, `HOST`, `PASSWORD` must be added

Once we're done with our settings, we can [[01 CLI#Create an empty dev database|Create an empty dev database]]

### Other Settings
- `TIME_ZONE` to set your time zone
- `INSTALLED_APPS` setting at the top of the file
	- holds the names of all Django applications that are activated in this Django instance
	- We can add a reference to our app's config class here (see [[06 Models#Activating / Modifying Models|Activating Models]]) to link our app with our environment
	- `django.contrib.staticfiles` in `INSTALLED_APPS` serves as a single location that collects [[09 Templates and Static Files#Static Files|Static Files]]
- `TEMPLATES`: how Django will load and render [[09 Templates and Static Files|Templates]]
	- Default settings file configures a `DjangoTemplates` backend whose `APP_DIRS` option is set to True
	- By convention DjangoTemplates looks for a “`templates`” subdirectory in each of the `INSTALLED_APPS`
- ` STATICFILES_FINDERS`: Contains a list of finders that know how to discover static files from various sources
	- One of the defaults is `AppDirectoriesFinder` which looks for a “static” subdirectory in each of the `INSTALLED_APPS` 

