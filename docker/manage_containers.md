---
layout: page
title: "Managing Docker Containers"
permalink: /docker/manage_containers
---

{% include breadcrumbs.html %}

## ---------

## List containers on your system

`docker ps` will list all running containers `docker ps -a` will list all running and stopped/exited containers.

## Starting, stopping and sending signals to containers

To stop a container `docker stop <container_name or id>` - will SIGTEM then SIGKILL the main process but containers resources remain so it can be restarted.

Once stopped you can restart/continue with `docker start <container_name or id>`

You can also use `docker kill <container_name or id>` which will send the SIGKILL signal by default; but can be combined with `--signal` option to send any signal to the main process in the containers. For example `docker kill --signal=SIGHUP my_container.

## Removing containers

`docker rm <containers_name || container_id>` for example `docker rm web-server`

To remove a containers and its volumes use the `--volumes` option `docker rm --volumes nginx`

A container needs to be stopped to be removed. You can use the `--force` flag to remove a running container.

### Bulk remove

`docker rm $(docker ps --filter status=exited -q)` or using `xargs` `docker ps --filter status=exited -q | xargs docker rm`

## Inspecting a containers

`docker inspect <container-name>` will give you JSON output with all the data about a container.
