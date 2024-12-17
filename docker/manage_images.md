---
layout: page
title: "Managing Docker Images"
permalink: /docker/manage_images
---

{% include breadcrumbs.html %}

## ---------

## List locally cached mages

`docker image ls -a`

## Deleting locally cached images

Delete a single image

`docker image rm <image_id>` or docker image rm `docker image rm <name:version>`

You can also use `docker rmi` as a shorthand for `docker image rm`.

To delete all local images

`docker rmi -f $(docker images -aq)` The `-q` option just outputs the id of the image.

