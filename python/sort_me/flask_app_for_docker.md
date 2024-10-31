---
layout: page
title: "Python: Demo Flask App For CI Tutorial"
permalink: /sort_me/flask_ci_demo
---

[comment]: <> (TODO: Go over this page and see how the Github links are working if they don't align with the steps use the code from the tutorial on this page and just link to the full project you have in GitHub.)

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

If you want to get the details for the redis container you can use the command `docker inspect redis-server` which will give you a json of all the data about the container.

## Connecting to Redis From Python

```python
>>> from redis import Redis
>>> redis = Redis()
>>> redis.incr("page_views")
3
>>> redis.incr("page_views")
4
```

Example above will connect to default Redis por on localhost. If you want to be explicit you can use `redis = Redis(host="127.0.0.1", port=6379)`. You can also use `redis = Redis.from_url("redis://localhost:6379/")`.

## Implement and Run the Flask Application Locally

Code for the flask application can be found [here](https://github.com/xcomfan/real_python_examples/blob/main/page-tracker/src/page_tracker/app.py)

To run the code locally; with the environment sources use the command `flask --app page_tracker.app run`.

If you want to be able to test from another computer on the same network you need to bind the app to 0.0.0.0. You can also enable debut and set the port.  Example command for that is `flask --app page_tracker.app run --host=0.0.0.0 -port=8080 --debug`.

Once the app is running you can use your browser or curl to navigate to `http://127.0.0.1:5000` and see the page tracking in action.

### Test and Secure Your Web Application

#### Cover the Source Code With Unit Tests

Pytest is more popular than unittest which is bundled in the standard library so we will be using that.

We add it to the [toml file](https://github.com/xcomfan/real_python_examples/blob/main/page-tracker/pyproject.toml)

The change we are making we are grouping [optional dependencies](https://setuptools.pypa.io/en/latest/userguide/dependency_management.html#optional-dependencies) under a common name (`dev` in this case). Below is the relevant snippet of the toml file

```toml
...
[project.optional-dependencies]
dev = [
    "pytest",
]
...
```

Once we update teh toml file, we need to reinstall our Python package with the optional dependencies using the command `python -m pip install --editable ".[dev]"`. You can use the square brackets to list the optional dependencies you want to install. The reason they are in quotes is a best practice to prevent the shell from potentially doing file expansion.

Since we are following the `src` layout in the project, we don't have to keep the test modules in the same folder or event he same package as the code we are testing.  Our project tree should look like.

```text
page-tracker/
│
├── src/
│   └── page_tracker/
│       ├── __init__.py
│       └── app.py
│
├── test/
│   └── unit/
│       └── test_app.py
│
├── venv/
│
├── constraints.txt
└── pyproject.toml
```

Here we take a bit of a detour into testing, but TLDR is we need to refactor our app.py code to use dependency injection so that we can mock the Redis server.  The changes are in the [accompanying repository](https://github.com/xcomfan/real_python_examples/blob/41aa779c519273108d0530e9fa0d5234de3dfd2b/page-tracker/src/page_tracker/app.py).

To execute your [unit tests](https://github.com/xcomfan/real_python_examples/blob/7f5fd2a97326b1f9dbd2566fd0cf100b9b7b0c15/test/unit/test_app.py) use the command `python -m pytest -v test/unit/`

#### Check Component Interactions Through Integration Tests

Goal of integration tests is to check how your components interact with each other as part of a larger system. For this page tracker example the integration tests will check if we can communicate with the Redis server.

We will be adding the `pytest-timeout` plugin to simulate a failed connection. By updating the toml file as seen [here](https://github.com/xcomfan/real_python_examples/blob/d636a98d4408f0b42bcc4307d7dddc041f7d30e9/page-tracker/pyproject.toml)

Once we make the update we need to re-install our package with the optional dependencies via `python -m pip install --editable ".[dev]"

Now we will add `conftest.py` to keep common fixtures, and the subfolder for unit tests. This will make the project structure look like...

```text
page-tracker/
│
├── src/
│   └── page_tracker/
│       ├── __init__.py
│       └── app.py
│
├── test/
│   ├── integration/
│   │   └── test_app_redis.py
│   │
│   ├── unit/
│   │   └── test_app.py
│   │
│   └── conftest.py
│
├── venv/
│
├── constraints.txt
└── pyproject.toml
```

While our web application has just one component, we can think of Redis as another component. Therefore, an integration test might look similar to our unit test, except that the Redis client won't be mocked.

Integration test code can be found [here](https://github.com/xcomfan/real_python_examples/blob/7103a3ab6276cebd273967213ce9220549bd5c75/page-tracker/test/integration/test_app_redis.py)

The test function takes two fixtures as parameters:

1. A Redis client connected to a local data store
2. Flask's test client hooked up to our web application

To make the second one available to the integration test we will move the `http_client()` fixture from `test_app` module to [`conftest.py`](https://github.com/xcomfan/real_python_examples/blob/7103a3ab6276cebd273967213ce9220549bd5c75/page-tracker/test/conftest.py)

Because `conftest.py` is located one level up in the folder hierarchy, `pytest` will pick up all the fixture defined in it and make them visible throughout your nested folders.

For the Redis Client fixture we give it a scope of `module` to reuse the same Redis client fixture for all functions within a test module.

You can now run your integration test with the command `python -m pytest -v test/integration/`

To simulate the Redis server being unavailable we can stop the redis container and run our test.

```bash
docker stop redis-server
python -m pytest -v test/integration/
```

This exposes an issue where our code does not gracefully handle Redis connection errors. We will add an additional unit test [`test_should_handle_redis_connection_error`](https://github.com/xcomfan/real_python_examples/blob/94b785d12a5d68a7a0be8a7f89bce886777141ab/page-tracker/test/unit/test_app.py) and [fix the issue](https://github.com/xcomfan/real_python_examples/blob/94b785d12a5d68a7a0be8a7f89bce886777141ab/page-tracker/src/page_tracker/app.py).

In the test code we set the mocked `.incr()` method [side effect](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.side_effect) so that method will raise the `redis.ConnectionError` exception which is what we saw when the integration test failed.

#### Test a Real World Scenario End to End (E2E)

Since we are trying to test the full stack in an E2E test we will need a deployment environment that mimics the production environment as closely as possible. You would use your E2E testing as part of a continuous integration pipeline.

Adding the E2E tests our project structure now looks like...

```text
page-tracker/
│
├── src/
│   └── page_tracker/
│       ├── __init__.py
│       └── app.py
│
├── test/
│   ├── e2e/
│   │   └── test_app_redis_http.py
│   │
│   ├── integration/
│   │   └── test_app_redis.py
│   │
│   ├── unit/
│   │   └── test_app.py
│   │
│   └── conftest.py
│
├── venv/
│
├── constraints.txt
└── pyproject.toml
```

The main difference between this E2E test and the integration test we did earlier is that we will be sending an actual HTTP request though the network to a live web server. Instead of relying on Flask's test client we will use the `requests` library which we need to add to our [`pyproject.toml`](https://github.com/xcomfan/real_python_examples/blob/d11c7be4114cba61b549114336302bb1413e47f0/page-tracker/pyproject.toml) file as another optional dependency.

As before after adding the `requests` dev dependency re-install your project package with the command `python -m pip install --editable ".[dev]"` and create the [`.../test/e2e/test_app_redis_http.py`](https://github.com/xcomfan/real_python_examples/blob/d11c7be4114cba61b549114336302bb1413e47f0/page-tracker/test/e2e/test_app_redis_http.py) test file.

The test function here receives the `flask_url` as an argument which `pytest` injects as a fixture. 

The other change we need to make is to our app because currently it expects the Redis server to always be running on localhost. The changed version can be seen [here](https://github.com/xcomfan/real_python_examples/blob/d11c7be4114cba61b549114336302bb1413e47f0/page-tracker/src/page_tracker/app.py).

To extend pytest with custom command line arguments for specifying the Redis URL you need to update [`conftestpy`](https://github.com/xcomfan/real_python_examples/blob/d11c7be4114cba61b549114336302bb1413e47f0/page-tracker/test/conftest.py).

To run the E2E test make sure that your Redis docker container is running. If its not you can start it with `docker start redis-server`.

You also need to have the Flask application running which you can start with the command `flask --app page_tracker.app run`

To run the integration test you use the command `python -m pytest -v test/e2e --flask-url http://127.0.0.1:5000 --redis-url redis://127.0.0.1:6379`.

### Perform Static Code Analysis and Security Scanning

We will add the following tools to our [`pyproject.toml`](https://github.com/xcomfan/real_python_examples/blob/67dddd8c86d98482896c0b8720687a6165c4ce33/page-tracker/pyproject.toml) file.

* [bandit](https://bandit.readthedocs.io/en/latest/) - vulnerability scanning
* [black](https://black.readthedocs.io/en/stable/) - find formatting inconsistencies
* [flake8](https://flake8.pycqa.org/en/latest/) - checks for [PEP8 compliance](https://realpython.com/python-pep8/)
* isort - organize import statements according to [official recommendation](https://peps.python.org/pep-0008/#imports)
* pylint

Without the `--check` flag both `black` and `isort` will auto fixe the file for you.

```bash
python -m black src/
python -m isort src/
python -m flake8 src/
# once you believe everything is clean try pyline
python -m pylint src/
python -m bandit -r src/ # bandit needs -r if scanning directory
```

As usual after updating our dependencies we need to re install our package and update the [`constraints.txt`](https://github.com/xcomfan/real_python_examples/blob/67dddd8c86d98482896c0b8720687a6165c4ce33/page-tracker/constraints.txt) file.  We can do that with the following commands.

```bash
python -m pip install --editable ".[dev]"
python -m pip freeze --exclude-editable > constraints.txt
```

side note difference between `0.0.0.0` and `127.0.0.1` is `0.0.0.0` will bing to public while `127.0.0.1` is bound to localhost and not exposed outside the system.

## Dockerize Your Flask Web Application

### Creating the Dockerfile

we will add the `Dockerfile` in the project root folder at th same level as `src/`. You can technically name this file anything you want but sticking to convention will save you having to manually specify the file each time.

```text
page-tracker/
│
├── src/
│   └── page_tracker/
│       ├── __init__.py
│       └── app.py
│
├── test/
│
├── venv/
│
├── constraints.txt
├── Dockerfile
└── pyproject.toml
```

The `Dockerfile` needs to have a specific format documented [here](https://docs.docker.com/reference/dockerfile/)

We are using the [official Python image](https://hub.docker.com/_/python) as our base image. In our case the `slim-bullseye` name at end of the image name means that we are using the slimmed down variant of `Debian Bullseye`

The `USER` and `WORKDIR` directives makes it so that code runs as the `realpython` user and the current working directory is set to that users home directory.

Because Linux systems need a global Python installation we still need to set up a virtual environment in our container to avoid version conflicts. We do that with the following lines.

Here we set an environment variable `VIRTUALENV` and install our virtual environment there. Instead of activating the virtual environment we update the `PATH` variable. The reason for this is activating the environment int he typical way would be only temporary and wouldn't affect Docker containers derived from your image. Furthermore if you activated the environment via the docker `RUN` command it would only last until the next instruction in your Dockerfile because each starts a new session.

When installing the dependencies and your project package in a Dockerfile you want to split those two steps to take advantage of layer caching as the dependencies will change less frequently than your package.

Its a good practice to install just the `project.toml` and `constraints.txt` files into your containers instead of your project directory. This helps you keep a slimmer image and makes it easier to take advantage of container layering.

Before you install your dependencies into the containers you want to make sure tup update pip. Its possible that some packages will not install if your version of pip is outdated. You can combine the pip upgrade along with installing your dependencies into a single `RUN` command.

We upgrade `pip` and `setuptools` to their most recent versions. Then we install the third party libraries that our project requires, including the optional dependencies for development. We constrain them to make sure the versions are consistent. We also disable `pip` caching with the `--no-cache-dir` as we won't need them outside our virtual environment in the container so there is no need to cache them. This makes your docker image smaller.

Because we install dependencies without installing our `page-tracker` package in the Docker image they will stay in a cached layer and thus any changes to our source code won't require re-installing those dependencies.

We will run our tests and linters and static analysis as part fot he build process.

We copy the `src/` and `test/` folders from our host machine then we install the `page-tracker` package into the virtual environment. By baking the automated testing tools into the build process, you ensure that if any one of them returns a non-zero exit status code, then building your Docker image will fail. That is what you want when implementing a continuous integration pipeline. Note that we are disabling low-sensitivity `pylint` issues `C0114`, `C0116` and `R1705` which are of little importance now.

At this point we just run the unit tests because we don't have the Redis containers available to run the integration and end to end tests.

The last step is tot ell the containers what command to run. At this point we will have the containers start using the Flask built in development server. Note that we are binding the host to the `0.0.0.0` address in order to make our application accessible from outside the Docker container.

Putting it all together we git [this Dockerfile](https://github.com/xcomfan/real_python_examples/blob/0935c4472276a1e5f9899f7a54c182b429db5b73/page-tracker/Dockerfile) and you can build it with the command `docker build -t page-tracker .`. This command will look for Docker file in the current directory `.` and tag the resulting image with the default label `latest` so the full image name will be `page-tracker:lates`.

### Reorganize Your Dockerfile for Multi-Stage Builds

The idea behind [multi-stage builds](https://docs.docker.com/build/building/multi-stage/) is to partition your Dockerfile into stages, each of which can be based on a completely different image. This is particularly useful when your application's development and runtime environments are different. For example, you can install the necessary build tools in a temporary image meant just for building and testing your application and then copy the resulting executable into the final image. Multi stage builds can make your images much smaller and more efficient.

To start with the refactor we will make a copy of `Dockerfile` called `Dockerfile.dev`. Note that when running docker build you specify the file you want to use for the build with the `-f` option as in `docker build -f Dockerfile.dev -t page-tracker .`  We are keeping `Dockerfile.dev` for later and making our changes for the multi stage build in `Dockerfile`.

Each stage in a Dockerfile begins with its own `FROM` instructions, so we will have two. The first stage will be nearly identical to our current `Dockerfile` we copied from to create `Dockerfile.dev` except that we give the stage the name `builder` which we refer to later.  Because we will be transferring your packaged page tracker application from one image to another, we need to add the extra step of building a distribution package using the `Python wheel` format. The `pip wheel` command will create a file named something like `page_tracker-1.0.0-py3-none-any.whl` in the `dist/` subfolder. We also remove the `CMD` instruction from this stage, as it'll become part of the next stage.

The second and final stage, implicitly named `stage-1`, looks a little repetitive because its based on the same image. In this second stage we start with the familiar steps of upgrading system package and creating a user and making a virtual environment. Then, we copy our wheel file which was generated in the prior `builder` stage and install it with `pip` as before. We also bring back the `CMD` command to start our application. In this setup the first stage is responsible for installing all the dependencies and running tests as well as generating the wheel file. The next (`stage-1`) stage just has to copy the finished wheel. Also not that the `builder` stage is temporary, so there will be no trace of it in your Docker mages afterward.

The refactored for multi stage builds Dockerfile should look like [this](https://github.com/xcomfan/real_python_examples/blob/59676ca2a15cdaa8e66fcd4583340ba7eb0eccc4/page-tracker/Dockerfile)

### Build and Version Your Docker Image

It is a good idea to have a versioning scheme and to tag your builds so that you are able to rollback if needed.

Some strategies for versioning are 

* [Semantic versioning](https://semver.org) - 3 numbers delimited with a dot to indicate the major, minor and patch versions.

* Git commit hash - uses the SHA-1 hash of Git commit tied to the source code in your image.

* Timestamp - uses temporal information, such as Unit time to indicate when the image was built.

You can also combine these strategies.

In this tutorial we will use the Git commit hash approach.

As a side note, you can get a good start point for a Python project `.gitignore` file via `curl -sL https://www.gitignore.io/api/python,pycharm+all > .gitignore`

You can get the hash of the current commit via `git rev-parse HEAD` or if you want the short version `git rev-parse --short HEAD`

We can use the commit hash to tag our docker image using the command. `docker build -t page-tracker:$(git rev-parse --short HEAD) .`

### Push the Image to a Docker Registry

You can use a managed service docker registry such as the one you get from AWS. You can also host your own with an [open source distribution continer](https://hub.docker.com/_/registry)

Docker hub offers a free tier of their managed service where you get unlimited public and one free private repository.

An image repository on your Docker Hub account is a collection of Docker image that uses can upload or download. Each repository can contain multiple tagged versions of the same image. In this regard, a Docker Hub is analogous to a GitHub repository but tailored for docker images rather than code.

Once you set up your Docker account use `docker login -u <username>` to login from command line. If you are using 2FA you need to set up an access token and provide that as your password.

Docker hub controls what repository your image is pushed to via tagging. You tag the image using your Docker Hub username and repository as a prefix. For example `docker tag page-tracker:719e61c xcomfan/page-tracker:719e61c`. This form we are follwing is `registry/username/repository:tag`. The registry part can be left out when you are pushing to default docker hub as we are doing in this example.  Otherwise it can be a domain address or an IP address with an optional port number of your private registry instance. If you don't provide a tag Docker implicitly applies the `latest` tag. You can tag the same image with more than one tag for example `docker tag page-tracker:719e61c xcomfan/page-tracker:latest`

Once you've correctly tagged the images, you can send them to the desired registry with `docker push` for example...

```bash
docker push xcomfan/page-tracker:719e61c
docker push xcomfan/page-tracker:latest
```

Note that you are not pushing the same image twice. Docker will recognize they are the same and only upload the meta data needed. If all worked you should see your images on Docker Hub.

To add collaborators you can used paid features or generate read only tokens for those who need to pull down your images.

At this point if you were to hop on another computer you can get the image via the command `docker pull xcomfan/page-tracker`. As this command does not specify any tag for the image Docker will pull the one tagged with `latest`. You don't need to manually pull images. Docker will pull them automatically when you try running them for example if you were to run the command `docker run -p 80:5000 --name web-sevice xcomfan/page-tracker`.

Adding the `-rm` flag will automatically remove the container when it stops if you don't want to do this by hand.

To see what networks you have defined locally (this is in the context of getting the container we created connected to Redis container) you can use the command `docker network ls` and if you need to create the page tracker network as we did before you would use `docker network create page-tracker-network`.

You can create a volume for your Redis container with `docker volume create redis-volume`.

After stopping and removing the old redis container start the new one with the volume attached with. `docker run -d -v redis-volume:/data --network page-tracker-network --name redis-service redis:7.0.10-bullseye`

And you can start our flask application container using.

`docker run -d -p 80:5000 -e REDIS_URL=redis://redis-service:6379 --network page-tracker-network --name web-service xcomfan/page-tracker`

* `-d` flag is for running the containers in the background (detached) without a terminal.

* `p` flag is for port forwarding

* `-e` flag is for environment variables

You can now access your application at `http://localhost` via browser or `curl`

## Orchestrate Containers Using Docker Compose

[Docker Compose](https://docs.docker.com/compose/) is a tool that works on top of Docker and simplifies running multi-container Docker applications.

If you are using Docker Desktop; Docker Compose is bundled with it.

### Define a Multi Container Docker Application

Because we are defining a multi container Docker applications that could potentially grow to include many more services, its worthwhile to re-arrance the folder structure of our project. We will create a new subfolder called `web` in the root of the project where all the files related to our Flask web service will live. The virtual environment directory belongs to this new subfolder because other services might be implemented in a different language such as Jva or C++. 

You can't just move the virtual environment folder as that would break the script in the virtual environment. You need to deactivate the environment remove the directory and create a new one.

```bash
deactivate
cd page-tracker/
rm -rf venv/
python3 -m venv web/venv/ --prompt page-tracker
source web/venv/bin/activate
 python -m pip install --upgrade pip
```

We also add at the project root a `docker-compose.yml` file. The [Compose File](https://docs.docker.com/reference/compose-file/) is where you define services, networks, and volumes.  The complete files for our example is [here](https://github.com/xcomfan/real_python_examples/blob/d9a84880b09b85681b667072cea8e0a3f871e626/page-tracker/docker-compose.yml)

Few notes on the file

* You are able to scale up the number of each of the containers.

* Some values in the configuration file are quoted, while others aren't. This is a precaution against a in older YAML format specification which treats certain characters as special if they appear in unquoted strings.

To run the docker compose file first we need to stop all of our running containers (pertinent to this app).

```bash
docker stop -t 0 web-service redis-service
docker container rm web-service redis-service
docker network rm page-tracker-network
docker volume rm redis-volume
``` 

Also delete any local images...


```bash
$ docker images
REPOSITORY                TAG              IMAGE ID       CREATED      SIZE
page-tracker              dde1dc9          9cb2e3233522   1 hour ago   204MB
page-tracker              latest           9cb2e3233522   1 hour ago   204MB
realpython/page-tracker   dde1dc9          9cb2e3233522   1 hour ago   204MB
realpython/page-tracker   latest           9cb2e3233522   1 hour ago   204MB
(...)

$ docker rmi -f 9cb2e3233522
Untagged: page-tracker:dde1dc9
Untagged: page-tracker:latest
Untagged: realpython/page-tracker:dde1dc9
Untagged: realpython/page-tracker:latest
Deleted: sha256:9cb2e3233522e020c366880867980232d747c4c99a1f60a61b9bece40...
```

If you really want to start from scratch and don't mind loosing data you can use `docker system prune --all --volumes` ***WARNING:*** this will remove everything created Docker.

To bring up your application via the Docker Compose file use the command `docker compose up -d`

The docker compose plugin provides several useful commands for managing your multi container application. Few are demonstrated below.

```bash
$ docker compose ps
NAME                           COMMAND                  SERVICE        ...
page-tracker-redis-service-1   "docker-entrypoint.s…"   redis-service  ...
page-tracker-web-service-1     "flask --app page_tr…"   web-service    ...

$ docker compose logs --follow
(...)
page-tracker-web-service-1    |  * Running on all addresses (0.0.0.0)
page-tracker-web-service-1    |  * Running on http://127.0.0.1:5000
page-tracker-web-service-1    |  * Running on http://172.20.0.3:5000
page-tracker-web-service-1    | Press CTRL+C to quit

$ docker compose stop
[+] Running 2/2
 ⠿ Container page-tracker-web-service-1    Stopped                     10.3s
 ⠿ Container page-tracker-redis-service-1  Stopped                      0.4s

$ docker compose restart
[+] Running 2/2
 ⠿ Container page-tracker-redis-service-1  Started                      0.4s
 ⠿ Container page-tracker-web-service-1    Started                      0.5s

$ docker compose down --volumes
[+] Running 4/4
 ⠿ Container page-tracker-web-service-1    Removed                      6.0s
 ⠿ Container page-tracker-redis-service-1  Removed                      0.4s
 ⠿ Volume page-tracker_redis-volume        Removed                      0.0s
 ⠿ Network page-tracker_backend-network    Removed                      0.1s
```

### Replace Flask's Development Web Server With Gunicorn

As we did before with the Redis container to drop into the CLI; we will use Docker's ability to provide an alternative run command to a container to run Flask through a production grade web server.

We are going to be using [Gunicorn](https://gunicorn.org/) which is a pure-Python implementation of the Web Server Gateway Interface(WSGI) protocol. To start using it we must first add the `gunicorn` package as another dependency in our project by updating the [`web/pyproject.toml` file](https://github.com/xcomfan/real_python_examples/blob/1f1a1a0a0f1943d8fb3fbaebeb24b9a7ad9933b9/page-tracker/web/pyproject.toml). We add `gunicorn` as a regular dependency.

As usual we need to re-install the `page-tracker` package locally to get the update.

```bash
python -m pip install --editable "web/[dev]"
 python -m pip freeze --exclude-editable > web/constraints.txt
```

Now that we have installed `gunicorn` we can start using it. We will add a new `command` attribute under the `web-service` key in our [`docker-compose.yml` file](https://github.com/xcomfan/real_python_examples/blob/c3146b30314c9cb882d6eb16c6be8afb87138aa5/page-tracker/docker-compose.yml). This command will take precedence over your Dockerfile's default command, which relies on Flask's development server. To show the difference we are running the server on port 8000 instead of 5000, so we also need to change the port mapping.

After making the Docker compose changes you need to rebuild. You can do this with either `docker compose build` or combine the operations with `docker compose up --build -d`.

## Run End to End Tests Against the Services

### Running locally on your system.

In the first attempt we will execute our end to end test locally from our machine. For this to work all the services. This is actually not ideal because you don't want to expose certain services to the public. We make the change in [`docker-compose.yml`](https://github.com/xcomfan/real_python_examples/blob/79b27a359178fab2c9162fdd162530b1cfd15e2d/page-tracker/docker-compose.yml) to expose the Redis port `6379`.

If there is an existing Docker container for `redis-service`, then we will need to remove that container first, even if its currently stopped, to reflect the new port forwarding rules. Fortunately Docker Compose will automatically detect the changes in `docker-comose.yml` file and re-create your containers as needed.

Run the following commands

```bash
$ docker compose up -d
[+] Running 2/2
 ⠿ Container page-tracker-redis-service-1  Started                      1.0s
 ⠿ Container page-tracker-web-service-1    Started                      1.2s

$ docker compose ps
NAME                           ...   PORTS
page-tracker-redis-service-1   ...   0.0.0.0:6379->6379/tcp
page-tracker-web-service-1     ...   0.0.0.0:80->8000/tcp
```

The `docker compose ps` command shows us the port forwarding.

You an now run your test with the command `python -m  pytest web/test/e2e --flask-url http://localhost --redis-url redis://localhost:6379`

If you want to simulate a failure you can temporarily puse and resume your containers with the commands below

```bash
docker compose pause
docker compose unpause
```

### Running from another container

A better option than running locally on your system and exposing Redis ports publicly is to run the tests from another container on the same network.

You can create this container manually, but a better option is to use Docker Compose [profiles](https://docs.docker.com/compose/profiles/) which can be activated on demand.

We will update our [`docker-compose.yml` file](https://github.com/xcomfan/real_python_examples/blob/c215b73cb6b9db6d1d7e761dfad61570e680f759/page-tracker/docker-compose.yml) to remove the Redis port forwarding we added and add a new service based on our old [`Dockerfile.dev` file](https://github.com/xcomfan/real_python_examples/blob/c215b73cb6b9db6d1d7e761dfad61570e680f759/page-tracker/web/Dockerfile.dev).

A few notes on the `docker-compose.yml` changes.

* Because Docker Compose has access to your host machine shell it will try to interpolate any reference or environment variables such ash `$REDIS_URL` or `$FLASK_URL` which appear in the `docker-compose.yml` file. These variables are most likely not defined when you run Docker Compose. To disable premature substitution of environment variables by Docker Compose, you escape the `$` with two dollar signs `$$`. This produces literal stings `$REDIS_URL` and `$FLASK_URL` in the command that will be executed in the resulting containers.

* When you start a multi-container application with Docker compose, only the core services that don't belong to any profile start. If you also with to start the services that were assigned to one or more profiles, then you must list those profiles using the `--profile` option.

`docker compose --profile testing up -d`

You can use the command `docker compose ps -a` to see that status. Notice that `page-tracker-test-service` existed with a 0 status code. To see the logs of the test you can run `docker compose logs test-service`

## Define a Docker-Based Continuous Integration Pipeline

To introduce continuous integration in your project you need the following

* Version control system
* Branching strategy
* Build automation
* Test automation
* Continuous integration server
* Frequent integrations

Some of the source control branching modesl are 

* [Trunk-Based Development](https://trunkbaseddevelopment.com/)
* [GitHub Flow](https://docs.github.com/en/get-started/quickstart/github-flow)
* [Forking Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/forking-workflow)
* [Release Branching](https://martinfowler.com/articles/branching-patterns.html#release-branch)
* [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)

For our application these are the steps we will be following

1. Fetch the latest version of the mainline to your computer.
2. Create a feature branch from the mainline.
3. Open a pull request to get early feedback from others.
4. Keep working on your feature branch.
5. Fetch the mainline often, merging it into your feature branch and resolving any potential conflicts locally.
6. Build, lint, and test the code on your local branch.
7. Push your changes whenever the local build and tests succeed.
8. With each push, check the automated tests that run on the CI server against your feature branch.
9. Reproduce and fix any identified problems locally before pushing the code again.
10. Once you’re done, and all tests pass, request that one or more coworkers review your changes.
11. Apply their feedback until the reviewers approve your updates and all tests pass on the CI server after pushing your latest changes.
12. Close the pull request by merging the feature branch to the mainline.
13. Check the automated tests running on the CI server against the mainline with the changes from your feature branch integrated.
14. Investigate and fix any issues that may be found, for example, due to new updates introduced to the mainline by others between your last push and merging.

### GitHub Actions Terminology

Each **workflow** consists of a number of **steps** executed by a **runner**. There are two types of runners:

* **GitHub-Hosted Runners:** Ubuntu Linux, Windows, macOS
* **Self-Hosted Runners:** Servers that you own and maintain

In this tutorial we will use the Ubuntu Linux GitHub hosted runner.

Unless you specify otherwise jobs within one workflow will run on separate runners in parallel, which can be useful for speeding up builds. At the same time, you can make one job depend on other jobs. You can also enable dependency caching to speed things up.

Each step of a job is implemented by an **action** that can be either:

1. A custom shell command or a script
2. A GitHub Action defined in another GitHub repository

There are many predefined GitHub Actions, which you can brows and find on [GitHub Marketplace](https://github.com/marketplace?type=actions) provided and maintained by the community.

GitHub uses YAML format for configuring workflows and it looks for a special `.github/workflows/` folder in your repository's root directory, where you can define your workflows. Additional you can store in this directory configuration files or scripts that execute on a runner.

### Add GitHub Workflow Configuration

To our project we will append the following structure.

```text
page-tracker/
│
├── web/
│
├── .git/
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
├── .gitignore
└── docker-compose.yml
```

GitHub has a [GitHub web-based editor](https://docs.github.com/en/repositories/working-with-files/managing-files/editing-files) that provides schema validations and suggestions for the available GitHub Actions and Attributes.

The content of the ci.yml file can be found [here](https://github.com/xcomfan/real_python_examples/blob/9437b5a3a2977e348b9165c756c894dc68197640/.github/workflows/ci.yml)

Some notes on what we are doing in the file...

* First step we are doing is checking out the commit that triggered the workflow using the [actions/checkout](https://github.com/actions/checkout) GitHub action. Because GitHub Actions are really GitHub repositories in disguise, you can provide a Git tag or a commit hash after the `@` sign to choose a specific version of the action.

* Next step is to build Docker images for your web and test services before executing the end to end tests though Docker Compose. Instead of using an existing action we will run a shell command on the runner. We are using YAMLs multiline literal folding `>` to break a long command into multiple lines for readability.  We request that Docker Compose rebuild our images with the `--build` flag and stop all containers when the `test-service` terminates. If you don't do this your job can run indefinitely.  

* These two steps will always run in response to the events listed at the top of the file, whic is either opening a pull request or merging a feature branch into the mainline. Additionally you want to push your new Docker image to Docker Hub when all the tests pass after successfully merging a branch into the mainline. Thus we run the next steps only when a push event triggers your workflow.

* We pass the credentials for logging into docker hub via a [secret context](https://docs.github.com/en/actions/learn-github-actions/contexts#secrets-context). You can define your secrets in your repository settings. While DockerHub secrets are encrypted if you try hard enough (shell command in one of your actions) you can see them.

* Once authenticated to Docker Hub, we tag and push our new Docker image using `docker/build-push-action` GitHub Actions from the marketplace. This action also runs conditionally when you merge a feature branch into the mainline.

* [GitHub Packages](https://github.com/features/packages) is another service integrated into GitHub that can act as a replacement for Docker Hub. Something worth exploring at some point.

### Enable Branch Protection Rules

In GitHub you can go to your repo setting and tune your branch protection rules. You can end your end to end test as a status check.

## Where to Go from Here

Some other features that you can practice setting up are...

* Automate deployment to the cloud for continuous delivery
* Configure persistent logging and monitoring of your services.
* Implement blue-green deployments
* Add feature toggles to experiment with canary releases and A/B testing