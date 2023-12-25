## Setup
After [[Python/Web Frameworks/Flask/00 Installation|Installation]] In project root, import Flask and create instance to Flask Object
A Flask application is an instance of the `Flask` class
- Everything about the application, such as configuration and URLs, will be registered with this class
- Doing so like below is not the best for larger projects, instead see [[11 Design Patterns#The application Factory|The application Factory]]
```python
from flask import Flask
app = Flask(__name__)
```
Then add function that returns content, in this case a string 

use Flask's `@app.route` decorator to map the URL route `/` to that function
The `route()` decorator to tell Flask what URL should trigger our function (see [[Python/Web Frameworks/Flask/03 Routing|Routing]])
```python
@app.route("/")
def home():
    return "Hello, Flask!"
```
- By default returns an HTML String
	- Can be Json, or HTML


