---
layout: page
title: "Docker Images"
permalink: /docker/images
---

## Docker images

There are some gotchas to keep in mind about Docker images.  

* Files that are removed by subsequent layers in the system are actually still present in the images; they are just inaccessible.  For example if layer A has a large file named BigFile and layer B removed the BigFile, layer C which builds on will still have the BigFile and that will have impact every time you need to publish or pull the image C even though the file is inaccessible.

* Each layer is an independent delta from the layer below it. Every time you change a layer, it changes every layer that comes after it. Changing the preceding layers means that they need to be rebuilt, re-pushed, and re-pulled to deploy your image to development. For example if you have a layer with source code and a layer that adds the needed libraries you want the source code layer which is more likely to change to be the subsequent layer.  In general you want to order your layers from least likely to change to most likely to change to optimize the image size for pushing and pulling.

* Secrets and images should never be mixed. Never put secrets into your images.

### Multistage image builds

[comment]: <> (TODO: Should move this to its own building section or best practices section once I have more Docker content)

One of the most common ways to unintentionally build large images is to do the actual program compilation as part of the construction of the application container image. This may leave development tools and artifacts in your image which can be quite large.  To resolve this issue Docker introduced **multistage builds**. With multi stage builds instead of producing a single image a Docker file can produce multiple images with each image considered a stage. Artifacts can be copied from preceding stages to the current stage.