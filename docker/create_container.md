---
layout: page
title: "Creating a Container From Registry Image"
permalink: /docker/create_container
---

{% include breadcrumbs.html %}

## ---------

## Download and run a container

`docker run -d --name  <name_you_want_to_call_it> <registry_image_name>`

for example

`docker run -d --name web_server nginx`

* `-d` flag specifies that the the containers should run in detached more (attached is the default)

`docker run -it  --rm <name_you_want_to_call_it> <registry_image>`

* `-i` `-t` flags (combined to `-it`) run the containers interactively.

* `--rm` flag tells Docker to remove the created container as soon as you terminate it.

## Exposing ports

`docker run -d --name=web_server -p 8080:80 -p 4443:443`

* `-p` is used to expose ports `8080` and `4443` and bind them to ports `80` and `443` for access on the host system.

## Adding volumes to a container

`docker run -d --name=web_server -p 8080:80 4443:443 -v www:/usr/share/nginx/html nginx`

* `-v` flag here mounts the volume `www` at the path `/usr/share/nginx/html` in the container. The volume `www` first needs to be created with the `docker volume create` command.

If you want to just mount a directory from the host system into the containers you would use the command similar to `docker run -d --name=web_server -p 8080:80 4443:443 -v /home/ubuntu/html:/usr/share/nginx/html nginx` This will bind the host directory `/home/ubuntu/html` to the container directory `/usr/share/nginx/html`.

## Adding environment variables to a containers

`docker run -d --name=web_server -p 8080:80 4443:443 -e NGINX_HOST=yourhost.com nginx`

## Controlling when a containers should restart

`docker run -d -p 80:80 443:443 --restart unless-stopped nginx`

* Your options for arguments to the `--restart` flag are
  * `--restart no`
  * `--restart on-failure`
  * `--restart always`
  * `--restart unless-stopped`