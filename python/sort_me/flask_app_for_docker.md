---
layout: page
title: "Python: Demo Flask App For CI Tutorial"
permalink: /sort_me/flask_ci_demo
---

## Sample Project Code

Code for the page tracker sample app can be found [here](https://github.com/xcomfan/real_python_examples/tree/main/page-tracker).

## Architecture Overview

We are using `pyproject.toml` to specify dependencies, and [setuptools](https://setuptools.pypa.io/en/latest/) as the [build backend](https://peps.python.org/pep-0517/).

The applications itself will be a Flask based app that uses Redis as a database to count the number of visits to the site. Redis and the Flask app will run in different containers.

## Generating Constraints File for Project Dependencies

Once the [`pyproject.toml`](https://github.com/xcomfan/real_python_examples/blob/main/page-tracker/pyproject.toml) file is created we can generate the `constraints.txt` file with the following commands.  

`python -m pip install --editable .` - This only ran once I upgraded pip to latest version in my virtual environment.
  
`python -m pip freeze --exclude-editable > constraints.txt`
  
The reasoning of these commands is that to generate to generate the constraints file you must first install your page-tracker project into the active virtual environment which will bring the required external libraries from PyPI. Even though we have not code yet, Python will recognize and install your package placeholder (we scaffolded the project with the below structure). We install the project in [editable mode](https://setuptools.pypa.io/en/latest/userguide/development_mode.html) for development. This allows us to make changes to source code and have them reflected in the virtual environment immediately without a re-install. In the second command we execute we are excluding the editable package from the constraints file.

```text
page-tracker/
│
├── src/
│   └── page_tracker/
│       ├── __init__.py
│       └── app.py
│
├── venv/
│
├── constraints.txt
└── pyproject.toml
```

With the constraints file that was generated anyone with the project can run `python -m pip install -c constraints.txt` and reproduce the same environment as you have. The `-c` option specifies that the pinned dependencies should be installed instead of the latest ones.

## Run a Redis Server Through Docker

[comment]: <> (TODO: Need to move the explanation stuff to docker section and just link to it.)

We will be using the [official Redis image](https://hub.docker.com/_/redis)

To start the image use the command `page-tracker % docker run -d --name redis-server redis` (obviously with docker set up on your system)

### Test the Connection to Redis

To test the connection to Redis we will start the same container as for Redis server, but use the `redis-cli` entry point instead of the default one.

When setting up multiple containers that need to work together, you should use [Docker Networks](https://docs.docker.com/network/)

First, create a new user defined [bridge network](https://docs.docker.com/network/bridge/) named after your project.

`docker network create page-tracker-network`

By defining a virtual network such as this, you can hook up as many Docker containers as you like and let them discover each other though descriptive names.

To list the networks in Docker use the command `docker network ls`

Next we connect our `redis-server` containers to the network we created with the command `docker network connect page-tracker-network redis-server`

Now we can run the second redis containers in CLI mode with the command...

`docker run --rm -it --name redis-client --link redis-server:redis-client redis redis-cli -h redis-server`.

The `--rm` flag tells Docker to remove the created container as soon as you terminate it. The `-i` `-t` flags (combined to `-it`) run the containers interactively. The `--network` option connects this containers to the virtual network we created earlier. This way both containers will receive hostnames corresponding to their names give by the `--name` option. By using the `-h` parameter tells Redis CLI to connect to a Redis server identified by its containers name.

Below is a session where we test the connection from Redis CLI.

```text
redis-server:6379> SET pi 3.14
OK
redis-server:6379> GET pi
"3.14"
redis-server:6379> DEL pi
(integer) 1
redis-server:6379> KEYS *
(empty array)
```

We can use port mapping to make Redis available outside the virtual network so that we can more easily develop/test our code.

To use port mapping we will stop the current Redis server and spin up a new one with a mapped port.

```bash
docker stop redis-server
docker rm redis-server
docker run -d --name redis-server -p 6379:6379 redis
```

## Connecting to Redis From Python

