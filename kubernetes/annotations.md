---
layout: page
title: "Kubernetes Annotations"
permalink: /kubernetes/annotations
---

## Annotations

Annotations provide a place to store additional metadata for Kubernetes object with the sole purpose of assisting tools and libraries.  They are a way for other programs driving Kubernetes via an API to store some data with an object. Annotations can be used for the tool itself, or to pass configuration information between external systems. While labels are used to identify and group objects; annotation are used to provide extra information about where an object came from, how to use it, or policy around that object.

When in doubt use an annotation and promote it to a label if there is need to use it as a selector.

Annotations are used to...

* Keep track of a "reason" for the latest update to an object
* Communicate a specialized scheduling policy to a specialized scheduler
* Extend data bout the last tool to update the resource and how it was updated (detect changes and do smart merge)
* Attach build, release, or image information that isn't appropriate for labels (Git hash, timestamp PR number etc)
* Enable the Deployment object to keep track of ReplicaSets that it is managing for rollouts
* Prototype alpha functionality in Kubernetes (instead of creating first class API filed, the parameters for that functionality are encoded in an annotation)

Annotations are used in various places in Kubernetes with the primary use case being rolling deployments. During rolling deployments, annotations are used to track rollout status and provide the necessary information required to roll back a deployment to a previous state.

## Defining annotations

Annotations keys use the same format as label keys but because they are often used to communicate information between tools the namespace part of the key is more important. Example keys would be `deployment.kubernetes.io/revision` or `kubernetes.io/change-cause`.

The value component of an annotation is a free-form field and there is no validation that any format is being followed. Its not uncommon to have a JSON document encoded as a string and stored as an annotation.

Annotations are defined in the common `metadata` section in every Kubernetes object.

```yaml
...
metadata:
  annotations:
    example.com/icon-url: "https://example.com/icon.png"
...
```
