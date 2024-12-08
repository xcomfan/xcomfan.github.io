---
layout: page
title: "Flask"
permalink: /python/web_frameworks/flask
---

## Installing

`python -m pip install Flask`

## Initiate Your Flask Project

[This code](https://github.com/xcomfan/real_python_examples/blob/1c0ad14d1892efe1ac89f8a5b5270511c1ed79a8/flask/rp_flask_board/app.py) is a hello world Flask project.

When you use a decorator such as `@app.route("/")`, the decorated function is referred to as a **view** in flask.

## Transform Your Project Into a Package

In this example our package folder will be called `board`. To convert the above hello world app to a package create a `board` directory (inside your project folder `rp_flask_board`) in this example and move the `app.py` file to `.../board/__init__.py`. 

You should now have a file structure that looks like.

```text
rp_flask_board/
│
└── board/
    └── __init__.py
```

With the new package structure to run your application execute the below command from the project folder (`rp_flask_board`)

`python -m flask --app board run --port 8000 --host 0.0.0.0 --debug`

## Work With an Application Factory

An **application factory** in Flask is a design pattern that is beneficial for scaling Flask projects. Using the application factory pattern lets you configure your Flask project flexibly as evolves making it easier to develop, test and maintain.

Instead of having your application's code at the root level of your `__init__.py` file, you work with a function that returns your application. To do so replace the content of `__init__.py` and make use [this content](https://github.com/xcomfan/real_python_examples/blob/f779d4a6570c3f2b9587905b425d6c8f4951c8e5/flask/rp_flask_board/board/__init__.py) instead.

This change will let you separate parts of you application such as routes, configurations and initialization into separate files.

## Leverage Blueprints

As you Flask project grows, you have lots of viw functions and other related bits of code that work together. Instead of adding all these directly to a basic app.py file, you can create **blueprints** that you register in your application factory. **Blueprints** are modules that contain related views that you can conveniently import in `__init__.py`.

As an example we will have a blueprint that stores the main pages of your project.

We create [pages.py](https://github.com/xcomfan/real_python_examples/blob/e2ab96e0471c553f1340834dcba0882860c6e36a/flask/rp_flask_board/board/pages.py) under the board folder.

In this code we create an instance of Blueprint named `bp` The first argument `pages` is the name of the blueprint. You will use this name to identify the blueprint in your Flask project. Second argument is the blueprint name which will be used to import pages into `__init__.py`.

Before you can visit the pages in the Blueprint you need to connect the `pages` blueprint with your Flask project. Make the following changes to [board/__init__.py](https://github.com/xcomfan/real_python_examples/blob/e2ab96e0471c553f1340834dcba0882860c6e36a/flask/rp_flask_board/board/__init__.py)

## Introduce Templates

Flask uses Jinja template engine and Jinja is installed as a dependency of Flask. Jinja supports template inheritance allowing you to create a base template that you extend in child templates.

To create a base template create the the file [`board/templates/base.html`](https://github.com/xcomfan/real_python_examples/blob/057061931bd426c8d51d6c2ad86e42a60a33b32a/flask/rp_flask_board/board/templates/base.html)

The base template in Jina is commonly named named `base.html`, and it contains the main HTML structure of your web pages. The `title` block gives you the opportunity to extend the title with a child template.

The code `block header` declares a `header` block. If a child template doesn't contain a header block then the `<header>` will stay empty.

The line `block content` Will either display "No messages" or the block content if there is any.

### Adding Child Templates

We create the child template for our home page which extends the base in the file [`board/templates/pages/home.html`](https://github.com/xcomfan/real_python_examples/blob/9f135520af488a4a6ccd2721c5f5e536f493bca4/flask/rp_flask_board/board/templates/pages/home.html)

The `extends` tag at the top of the file connects the child template with the parent template.

Child templates also contain `block` tags. By providing the block's name as an argument, you're connecting the blocks from the child template with the blocks from the parent template.

When you use `url_for()`, Flask creates the full URL to the given view for you. This means that if you change the route in your Python code, then the URL in your templates updates automatically.

Next we create the file [`board/templates/pages/about.html`](https://github.com/xcomfan/real_python_examples/blob/9f135520af488a4a6ccd2721c5f5e536f493bca4/flask/rp_flask_board/board/templates/pages/about.html)

Just like in `home.html` You add the page's title inside the header block and wrap it within `title`. Jinja will understand this structure and render the page's title in the `header` and `title` blocks.

With both the child templates in place, we need to adjust the views to return the templates instead of the plain strings. We update [pages.py](https://github.com/xcomfan/real_python_examples/blob/9f135520af488a4a6ccd2721c5f5e536f493bca4/flask/rp_flask_board/board/pages.py) and adjust the `home()` and `about()` views.

By default Flask expects your templates to be in a `templates/` directory, thus you don't need to include `templates/` in the path of the templates.

## Improving The User Experience

### Include a Navigation Menu

We want to have the navigation menu on every page so we add it to `base.html`. Instead of adding the navigation menu code directly to `base.html` we can leverage the `include` tag which loads the whole templates referenced by the tag into the position of the `include` tag. This lets you reuse the navigation menu code wherever it is needed.

**Included templates** are partials that contain a fraction of the full HTML code. To indicate that a template is meant to be included, you can prefix the name with an underscore `_`. We create the file [`templates/_navigation.html`](https://github.com/xcomfan/real_python_examples/blob/fd52bfde5ce645bec421837aa72fca65cd38421d/flask/rp_flask_board/board/templates/_navigation.html) for our navigation menu.

Note that `_navigation.html` does not contain an `extends` or `block` tag. We just focus on what needs to be in our navigation menu.

We then update our [`templates/base.html`](https://github.com/xcomfan/real_python_examples/blob/6da71e02c93206cb0d6833cc2e88f75c58520708/flask/rp_flask_board/board/templates/base.html) template to include the navigation.

Once you start your application you will see that both the home and about page now show the navigation menu. That's the power of template inheritance.

### Making Your Project Look Nice

Here we are going ot use CSS to style our content.

In Flask projects, you commonly save CSS files in `static/` directory. We create the file [`/board/static/styles.css](https://github.com/xcomfan/real_python_examples/blob/fd52bfde5ce645bec421837aa72fca65cd38421d/flask/rp_flask_board/board/static/styles.css)

With the style sheet file created we need to link to it in [`templates/base.html`](https://github.com/xcomfan/real_python_examples/blob/fd52bfde5ce645bec421837aa72fca65cd38421d/flask/rp_flask_board/board/templates/base.html)