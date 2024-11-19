---
layout: page
title: "Docker Quick Reference"
permalink: /docker/quick_ref
---

## Container lifecycle management

### Creating new container

#### From Docker Registry

`docker run -d --name  <name_you_want_to_call_it> <registry_image_name>`

`docker run --rm -it --name redis-client --link redis-server:redis-client redis redis-cli -h redis-server`

The `--rm` flag tells Docker to remove the created container as soon as you terminate it. The `-i` `-t` flags (combined to `-it`) run the containers interactively. The `--network` option connects this containers to the virtual network we created earlier. This way both containers will receive hostnames corresponding to their names give by the `--name` option. By using the `-h` parameter tells Redis CLI to connect to a Redis server identified by its containers name

### Start or stop a container

`docker start|stop <container_name> or <container_id>`

### List containers

For all containers use `docker ps -all` for just running ones you can use `docker ps`

## Remove stopped containers

`docker rm <container_name>`

## Connect to a running container

### Using docker exec

The `-i` makes the command executing interactive

`docker exec -i <container_id> /bin/bash`

### SSH way

use `docker ps` to see the containers you have running the use the command below to get IP address of container

`docker inspect -f "{{ .NetworkSettings.IPAddress }}" [container-name-or-id]`

once you have the ip you can log in with `ssh [username]@[ip-address]`

## Port Forwarding



## BREAKPOINT 


To use port mapping we will stop the current Redis server and spin up a new one with a mapped port.

```bash
docker stop redis-server
docker rm redis-server
docker run -d --name redis-server -p 6379:6379 redis
```

If you want to get the details for the redis container you can use the command `docker inspect redis-server` which will give you a json of all the data about the container.

```bash
docker stop redis-server
python -m pytest -v test/integration/
```

To run the E2E test make sure that your Redis docker container is running. If its not you can start it with `docker start redis-server`.

The `Dockerfile` needs to have a specific format documented [here](https://docs.docker.com/reference/dockerfile/)

We are using the [official Python image](https://hub.docker.com/_/python) as our base image. In our case the `slim-bullseye` name at end of the image name means that we are using the slimmed down variant of `Debian Bullseye`

Putting it all together we git [this Dockerfile](https://github.com/xcomfan/real_python_examples/blob/0935c4472276a1e5f9899f7a54c182b429db5b73/page-tracker/Dockerfile) and you can build it with the command `docker build -t page-tracker .`. This command will look for Docker file in the current directory `.` and tag the resulting image with the default label `latest` so the full image name will be `page-tracker:lates`.

The idea behind [multi-stage builds](https://docs.docker.com/build/building/multi-stage/) is to partition your Dockerfile into stages, each of which can be based on a completely different image. This is particularly useful when your application's development and runtime environments are different. For example, you can install the necessary build tools in a temporary image meant just for building and testing your application and then copy the resulting executable into the final image. Multi stage builds can make your images much smaller and more efficient.

To start with the refactor we will make a copy of `Dockerfile` called `Dockerfile.dev`. Note that when running docker build you specify the file you want to use for the build with the `-f` option as in `docker build -f Dockerfile.dev -t page-tracker .`  We are keeping `Dockerfile.dev` for later and making our changes for the multi stage build in `Dockerfile`.

Each stage in a Dockerfile begins with its own `FROM` instructions, so we will have two. The first stage will be nearly identical to our current `Dockerfile` we copied from to create `Dockerfile.dev` except that we give the stage the name `builder` which we refer to later.  Because we will be transferring your packaged page tracker application from one image to another, we need to add the extra step of building a distribution package using the `Python wheel` format. The `pip wheel` command will create a file named something like `page_tracker-1.0.0-py3-none-any.whl` in the `dist/` subfolder. We also remove the `CMD` instruction from this stage, as it'll become part of the next stage.

The second and final stage, implicitly named `stage-1`, looks a little repetitive because its based on the same image. In this second stage we start with the familiar steps of upgrading system package and creating a user and making a virtual environment. Then, we copy our wheel file which was generated in the prior `builder` stage and install it with `pip` as before. We also bring back the `CMD` command to start our application. In this setup the first stage is responsible for installing all the dependencies and running tests as well as generating the wheel file. The next (`stage-1`) stage just has to copy the finished wheel. Also not that the `builder` stage is temporary, so there will be no trace of it in your Docker mages afterward.

The refactored for multi stage builds Dockerfile should look like [this](https://github.com/xcomfan/real_python_examples/blob/59676ca2a15cdaa8e66fcd4583340ba7eb0eccc4/page-tracker/Dockerfile)

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

[Docker Compose](https://docs.docker.com/compose/) is a tool that works on top of Docker and simplifies running multi-container Docker applications.

If you are using Docker Desktop; Docker Compose is bundled with it.

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

We will update our [`docker-compose.yml` file](https://github.com/xcomfan/real_python_examples/blob/c215b73cb6b9db6d1d7e761dfad61570e680f759/page-tracker/docker-compose.yml) to remove the Redis port forwarding we added and add a new service based on our old [`Dockerfile.dev` file](https://github.com/xcomfan/real_python_examples/blob/c215b73cb6b9db6d1d7e761dfad61570e680f759/page-tracker/web/Dockerfile.dev).

A few notes on the `docker-compose.yml` changes.

* Because Docker Compose has access to your host machine shell it will try to interpolate any reference or environment variables such ash `$REDIS_URL` or `$FLASK_URL` which appear in the `docker-compose.yml` file. These variables are most likely not defined when you run Docker Compose. To disable premature substitution of environment variables by Docker Compose, you escape the `$` with two dollar signs `$$`. This produces literal stings `$REDIS_URL` and `$FLASK_URL` in the command that will be executed in the resulting containers.

* When you start a multi-container application with Docker compose, only the core services that don't belong to any profile start. If you also with to start the services that were assigned to one or more profiles, then you must list those profiles using the `--profile` option.

`docker compose --profile testing up -d`

You can use the command `docker compose ps -a` to see that status. Notice that `page-tracker-test-service` existed with a 0 status code. To see the logs of the test you can run `docker compose logs test-service`