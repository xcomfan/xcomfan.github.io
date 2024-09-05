---
layout: page
title: "Deploy Your Python Script on the Web With Flask"
permalink: /sort_me/flask_web_deploy
---

## Overview

This covers adapting an existing applications to the web not building a web application from ground up. This will require you to refactor the code into a web app, and deploy it to the internet.

## Brushing up on basics

### Python Code Distribution Types

Bringing your code to your users is called distribution. Traditionally there are 3 approaches you can use so that others can work with your programs. Python library, standalone program and Python web app.

### Web Applications

Web frameworks such as Flask help you get Python running on a website.

## Running a Web Application

### The HTTP Request-Response Cycle

Here is a generalized overview of what happens when a user interacts with a web applications

* Sending - User makes a request for a particular web page on your web app.
* Receiving - The request gets received by the web server that host your website.
* Matching - your web server now uses a program to match the user's request to a particular portion of your Python script.
* Running - The appropriate Python code is called up by that program.
* Delivering - The program then delivers this response back to your user though the web server.
* Viewing - Finally, the user can view the web server's response (in browser or otherwise)

The above process is called the HTTP Request-Response Cycle

### Hosting Your Site

To allow Flask to handle requests on the server side, you'll need to find a place where your Python code can live online. There are many such hosting providers but this course will use Google App Engine. Some other free options are PythonAnywhere, Repl.it and Heroku.

## Building a Basic Python Web App

### Setting up Your Project

Our sample project is called `hello-app` and it has the below file structure.

```text
hello-app/
|
|--- main.py
|--- requirements.txt
|--- app.yaml
```

`main.py` contains the Python code wrapped in a minimal implementation of the Flask web framework.
`requirements.txt` lists all of the dependencies your code needs to work properly.
`app.yaml` helps Google App Engine decide which settings to use on its server.

```python
# main.py

from flask import Flask

app = Flask(__name__)

@app.route("/")
def index():
    return "Congratulations, its a web app!"

if __name__ == "__main__":
    app.run(host="127.0.0.1", port=8080, debug=True)
```

```text
# requirements.txt

Flask==3.0.3
```

below is the content of the app.yaml file

```yaml
runtime: python3.10
```

### Testing Locally

To test locally you will need to set up a virtual environment and install `flask`. To set up a virtual environment run the following command from your project folder `python3 -m venv venv`. Your then source the virtual environment you created with the command `source venv/bin/activate`. To install the packages in your `requirements.txt` file (and any of their dependencies) run the command `python3 -m pip install -r requirements.txt`  When you are ready to test your app you can start it locally with the command `python main.py`. While your Flask app is running locally code changes should auto reload the app.

### Deploying to Google App Engine

Install the google cloud CLI, then run `gcloud init` and get the login sorted. The use `gcloud deploy` to deploy the app.

## Converting a Script to a Web App

We start with this basic Python script

```python
# convert.py

def fahrenheit_from(celsius):
    """Convert Celsius to Fahrenheit degrees."""
    try:
        fahrenheit = float(celsius) * 9 / 5 + 32
        fahrenheit = round(fahrenheit, 3)
        return str(fahrenheit)
    except ValueError:
        return "invalid input"
    
if __name__ == "__main__":
    celsius = input("Celsius: ")
    print("Fahrenheit:", fahrenheit_from(celsius))

```

Below we work the above script into our Flask based web app.  A few things to noe.

* The `<int:celsius>` syntax in the `@app.route` tells Flask to capture any test following the base URL and pass it on to the function the decorate wraps as the variable celsius. Note that `fahrenheit_from()` requires `celsius` as an input. Make sure the ULR path component you are capturing has the same name as the parameter you're passing to your function or Flask will give you an error. Adding `int:` before the variable name tells Flask to check whether the input it receives from the URL can be converted to an integer. If it can't Flask will display a `Not Found` error page. Because Flask is doing the type check for you, you can safely remove the try/except checks as only an integer will ever be passed to your code.

```python
# main.py

from flask import Flask

app = Flask(__name__)

@app.route("/")
def index():
    return "Congratulations, its a web app!"

@app.route("/<int:celsius>")
def fahrenheit_from(celsius):
    """Convert Celsius to Fahrenheit degrees."""
    
    fahrenheit = float(celsius) * 9 / 5 + 32
    fahrenheit = round(fahrenheit, 3)
    return str(fahrenheit)

if __name__ == "__main__":
    app.run(host="127.0.0.1", port=8080, debug=True)
```

## Improving the User Interface

In this exercise we will just use an HTML form element. We will use the index (`/`) path to return our HTML code.

A few notes...

* the HTML our index path is returning is not valid, but modern browsers are designed in a way where they can fill in the blanks.

* in the `<form`> element we set `action` to an empty string which makes browser direct the request to the same URL it was called from.

```python
# main.py

from flask import Flask

app = Flask(__name__)

@app.route("/")
def index():
    return """
    <form action="" method="get">
    <input type="text" name="celsius">
    <input type="submit" value="Convert">
    </form>
    """

@app.route("/<int:celsius>")
def fahrenheit_from(celsius):
    """Convert Celsius to Fahrenheit degrees."""
    
    fahrenheit = float(celsius) * 9 / 5 + 32
    fahrenheit = round(fahrenheit, 3)
    return str(fahrenheit)

if __name__ == "__main__":
    app.run(host="127.0.0.1", port=8080, debug=True)
```

## Receiving User Input

We need the functionality to be able to fetch the value being send to `index()` from our `<form>`. For this we will need to import Flask's `request` object. The `request` object contains the submitted value and gives you access to it through a dictionary syntax.

We don't trust user inputs, so we use Flask's built in `escape()` which converts HTML characters to their text equivalents. Larger applications would use Templates (see full Flask course for that).

```python
# main.py

from flask import Flask
from flask import request, escape

app = Flask(__name__)

@app.route("/")
def index():
    celsius = str(escape(request.args.get("celsius", "")))
    return (
        """
        <form action="" method="get">
        <input type="text" name="celsius">
        <input type="submit" value="Convert">
        </form>""" 
        + celsius
    )

@app.route("/<int:celsius>")
def fahrenheit_from(celsius):
    """Convert Celsius to Fahrenheit degrees."""
    
    fahrenheit = float(celsius) * 9 / 5 + 32
    fahrenheit = round(fahrenheit, 3)
    return str(fahrenheit)

if __name__ == "__main__":
    app.run(host="127.0.0.1", port=8080, debug=True)
```

## Finale web app

In this version we make some changes. Our prior version used only one URL endpoint, and because of this, you can't rely on Flask to do type checking via URL as in prior example. With this change we no longer need to use `escape()`. We can move input validation to `farenheit_from()`. Index will have a conditional statement added to check the value of celsius.

```python
# main.py

from flask import Flask
from flask import request

app = Flask(__name__)

@app.route("/")
def index():
    celsius = request.args.get("celsius", "")
    if celsius:
        fahrenheit = fahrenheit_from(celsius)
    else:
        fahrenheit = ""
    return (
        """
        <form action="" method="get">
        Celsius Temperature: <input type="text" name="celsius">
        <input type="submit" value="Convert">
        </form>""" 
        + "Fahrenheit: "
        + fahrenheit
    )

def fahrenheit_from(celsius):
    """Convert Celsius to Fahrenheit degrees."""
    try:
        fahrenheit = float(celsius) * 9 / 5 + 32
        fahrenheit = round(fahrenheit, 3)
        return str(fahrenheit)
    except ValueError:
        return "invalid input"

if __name__ == "__main__":
    app.run(host="127.0.0.1", port=8080, debug=True)
```
