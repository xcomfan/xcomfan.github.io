---
layout: page
title: "Docker Quick Reference"
permalink: /docker/quick_ref
---

# Container lifecycle management

## Start or stop a container

`docker start|stop <container_name> or <container_id>`

## List containers

For all containers use `docker ps -all` for just running ones you can use `docker ps`

## Remove stopped containers

`docker rm <container_name>`

# Connect to a running container

## Using docker exec

The `-i` makes the command executing interactive

`docker exec -i <container_id> /bin/bash`

## SSH way

use `docker ps` to see the containers you have running the use the command below to get IP address of container

`docker inspect -f "{{ .NetworkSettings.IPAddress }}" [container-name-or-id]`

once you have the ip you can log in with `ssh [username]@[ip-address]`