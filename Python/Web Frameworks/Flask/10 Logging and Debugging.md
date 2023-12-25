## Logging
Flask comes with a preconfigured logger
```python
app.logger.debug('A value for debugging')
app.logger.warning('A warning occurred (%d apples)', 42)
app.logger.error('An error occurred')
```

To see `logger.debug()` messages, you need to be in debug mode:
```python
...
app = Flask(__name__) 
app.debug = True
...
```


## Debugging
See [[02 CLI#Debugging|Debugging]]