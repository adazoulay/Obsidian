# Running
run the app by entering:
`python -m flask run`
* Runs the Flask development server
>[!tip]
>The dev server looks for `app.py` by default 

## Flags

### `--app`
- Need to tell the Flask where your application is with the `--app` option if the entry is not called `app.py`
	- Ex: `flask --app hello run`

### `--host` and `--port`
- To run the development server on a different IP address or port, use the host and port command-line arguments `--host=0.0.0.0 --port=80`
	- Ex: `python -m flask run --host=0.0.0.0 --port=80`.
	- Here, `--host=0.0.0.0` lists all public IPs 

### `--debug`
- Enabling `debug` mode allows for **hot reloading** and will show an **interactive debugger** in the browser if an error occurs during a request
	- To enable debug mode, use the `--debug` option
	- Ex:  `... flask run --debug`

# Debugging
To debug in VSCode:
- Start with setting a breakpoint in the code
- Switch to the `Run and Debug` view "⇧⌘D"
- If not already present, create a `launch.json` file containing a debug config
	- Option to create one should be at the bottom of the blue "Run and Debug" button
	- Select "Flask" from the dropdown menu
	- VSCode will populate the file for you
- Make sure the **Python: Flask** configuration is set in the dropdown (Top of panel, by green play/run button)
- Start the debugger by selecting the **Run** > **Start Debugging** menu command, or selecting the green **Start Debugging**

### `launch.json`
> - Configuration contains `"module": "flask",` which tells VS Code to run Python with `-m flask` when it starts the debugger
> - Defines the FLASK_APP environment variable in the `env` property to identify the startup file, which is `app.py` by default, but allows you to easily specify a different file
> - If you want to change the host and/or port, you can use the `args` array
