---
layout: page
title: "Data Visualization With Dash"
permalink: /python/sort_me/dash_data_vis.md
---

## What is Dash

Dash is an open sources framework for building data visualization interfaces. Dash uses Flask to create a web server, React.js to render the user interface and Plotly.js to generate charts. Dash takes care of making these technologies work together.

## Install

Naturally you want to do this in a virtual environment.

You can install Dash with pip using the command `pip install dash==2.0.0` (you don't have to pin the version just an example)

In the lesson we also installed Pandas so the full command was `pip install dash==2.0.0 pandas==1.3.3`

To actually get it running I had to downgrade `numpy` to version `1.26.4` and run the latest version of `dash` `2.18.1` and the above `pandas` `1.3.3`

## Building your app

Its kind of tricky to note the content so this is an example from the course to get you started. [avocado statistics dahs example](https://github.com/xcomfan/real_python_examples/tree/main/dash_intro)

## Running your app in dev mode

With the virtual environment sourced use the command `python app.py` (assuming app.py is your entry point) to launch your app in development mode. When running in the development mode changes made will be automatically reloaded.

## Deploying

Heroku is one option that the class covers, but use the docs and make it work for you if needed.
