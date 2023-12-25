In addition to the `request` object (See [[Python/Web Frameworks/Flask/05 Accessing Request Data|Accessing Request Data]] ) there is also a second object called `session` which allows you to store information specific to a user from one request to the next

This is implemented on top of [[Python/Web Frameworks/Flask/05 Accessing Request Data#Cookies|Cookies]] for you and signs the cookies cryptographically
- Read only unless they know the secret key

In order to use Sessions, you need to set a secret key.
```python
from flask import session

# Set the secret key to some random bytes. Keep this really secret!
app.secret_key = b'_5#y2L"F4Q8z\n\xec]/'

@app.route('/')
def index():
    if 'username' in session:
        return f'Logged in as {session["username"]}'
    return 'You are not logged in'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form method="post">
            <p><input type=text name=username>
            <p><input type=submit value=Login>
        </form>
    '''

@app.route('/logout')
def logout():
    # remove the username from the session if it's there
    session.pop('username', None)
    return redirect(url_for('index'))
```