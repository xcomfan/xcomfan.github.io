---
layout: page
title: "Kubernetes Volumes"
permalink: /kubernetes/volumes
---

## Using Volumes With Pods

To add a volume to a Pod manifest you need to add the `spec.volumes` section which defines all the volumes that may be accessed by containers in the Pod manifest. Not all containers are required to mount all of the volumes int the Pod. You also need to add `volumeMounts` array in the container definition which defines volumes mounted into a particular container and the path they are mounted at. Two different containers in a Pod can mount the same volume at different mount paths.

[comment]: <> (TODO: Should create some example here)

## What Kubernetes Volumes Can Be used For

### Communication/synchronization

tTwo containers can mount the same file system to use it as a communication/synchronization mechanism. One container can periodically pull down the latest code from Git and another container can serve that code content for example.

### Cache

Volume can be used as a cache while container is running. The volume will survive a restart so the cached content will persist until the container is deleted.

### Persistent Data

Sometimes a volume will be used for truly persistent data the life span of which is independent of the container using it. The volume should also be able to move from host to host. Kubernetes supports multiple options for this including NFS, iSCSI, EBS, Azure File Disk, etc. Usage of cloud provider persistent storage such as S3 and EBS requires plugins to Kubernetes from the specific cloud provider.

### Mounting the host filesystem

If you don't need a persistent volume but do need access to underlying file system for example access to /dev for devices. For these scenarios Kubernetes supports `hostPath` volumes which can mount arbitrary locations on the worker node into the container.

[comment]: <> (TODO: Secrets should be here once we get to that content)
