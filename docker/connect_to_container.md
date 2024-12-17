---
layout: page
title: "Connecting to Running Container"
permalink: /docker/create_container
---

{% include breadcrumbs.html %}

## ---------

## Using docker exec

The `-i` makes the command executing interactive

`docker exec -i <container_id> /bin/bash`

## SSH way

use `docker ps` to see the containers you have running the use the command below to get IP address of container

`docker inspect -f "{{ .NetworkSettings.IPAddress }}" [container-name-or-id]`

once you have the ip you can log in with `ssh [username]@[ip-address]`
