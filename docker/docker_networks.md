---
layout: page
title: "Docker Network"
permalink: /docker/network
---

This way both containers will receive hostnames corresponding to their names give by the `--name` option.

## What Is Docker Network?

When setting up multiple containers that need to work together, you should use [Docker Networks](https://docs.docker.com/network/)

## Creating User Defined Bridge Network

First, create a new user defined [bridge network](https://docs.docker.com/network/bridge/) named after your project.

`docker network create page-tracker-network`

By defining a virtual network such as this, you can hook up as many Docker containers as you like and let them discover each other though descriptive names.

## Managing User Defined Docker Networks

To list the networks in Docker use the command `docker network ls`

## Connecting a Container To a Docker Network

Next we connect our `redis-server` containers to the network we created with the command `docker network connect page-tracker-network redis-server`.

Once connected to the network you can start your containers per usual ``docker run --rm -it --name redis-client --link redis-server:redis-client redis redis-cli -h redis-server`.

