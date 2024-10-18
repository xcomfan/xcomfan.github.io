---
layout: page
title: "Kubernetes: Extensions"
permalink: /kubernetes/extensions
---

[comment]: <> (TODO: I stopped outlining this chapter as it became kind of har to follow without working though the examples. Will re pass later)

## Extending Kubernetes

Even cluster admins should be careful and use diligence when extending Kuberneetes with third party tools. Some extensions can be used as a vector to steal secrets or run malicious code. Extending a cluster makes it different than stock Kubernetes. When running multiple clusters, it is very valuable to build tooling to maintain consistency of experience across clusters which should include extensions that are installed.

### Points of extensibility

In addition to admission controllers and API extensions, there are a numbrer of ways to "extend" your cluster wihtout modifying the API server. These include DaemonSets that install automatic logging and monitoring tools that can scan your services for vulnerabilities.

Admission contorllers are called prior to the API object being written into the backing storage. Adminssion controllers can reject or modify API requests. There are several admission controllers that are built into the Kubernetes API server. The limit range admission controller sets default limits for Pods without default limits. Many systems use custom admission controllers to auto-inject sidecar continers into all Pods created on the system to create "auto-magic" experiences.

The other for of extension which can be used in conjunction with admission controllers is custom resources. With custom resources whole new API objects are added to the Kubernetes API surface area. These new APIs can be added to namespaces,are subject to RBAC and can be used with existing tools such as `kubectl`.

The first thing you need to do to create a custom resource is to create a CustomResourceDefinition. This objec tis a meta-resource (a resource that is the definition of another resource). As a concrete exmple consider defining a new resource to represent load tests in your cluster. When a new LoadTest resource is created, a load test is spun up in your Kubernetes cluster and drives traffic to a service.  The first steap in creating this resource is defining it through CustomResourceDefinition. Below is an example. The name of a custom resource has to have the form `<resource plural>.<api-group>` to ensure that each resource definition is unique to the cluster.  The API group in the spec musht match the suffix in the metadata. Once you use `kubectl create -f loadtest-resource.yaml` (loadtest-resource.yaml is the file name with below example) you can right away run the command `kubectl get loadtests`.

```yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: loadtests.beta.kuar.com
spec:
  group: beta.kuar.com
  versions:
    - name: v1
      served: true
      storage: true
  scope: Namespaced
  names:
    plural: loadtests
    singular: loadtest
    kind: LoadTest
    shortNames:
    - lt
```

Now that you have the resource created you can create the resource. The manifest for that is below. You can provide a schema for the CustomResourceDefinition, but unless you want to do validation you don't need to do that. Even if you do want to do validation you can register a validating admission controller. The manifest below can bre created using `kubectl create -f loadtest.yaml`

```yaml
apiVersion: beta.kuar.com/v1
kind: LoadTest
metadata:
  name: my-loadtest
spec:
  service: my-service
  scheme: https
  requestsPerSecond: 1000
  paths:
    - /index.html
    - /login.html
    - /shares/my-shares/
```

At this point we can list the loadtest resource but it does not do anything yet. You can use the CRUD API to create read update delete objects but they will take no action. To actually do something a controller needs to be present in the cluster to react to the new API we defined. We need a piece of code that will continuously monitor the custom resources and create, modify, or delete LoadTests as necessary to implement the API. There is a Watch API on the API server that your code can use to implement this control loop. The API is a bit hard to use so you should look into supported mechanisms such as `informer` in the [client-go library](https://pkg.go.dev/k8s.io/client-go/informers)

## Patterns for custom resources

### Just data

In the Just data pattern, you are simply using the API server for storage and retrieval of information for your application. You should not use the API server for storage and retrieval for your application; its not designed for that. API extensions should be used for control or configuration objects that help you manage the deployment or runtime of your application. An example use case for "just data" might be configuration for canary deployments of your application. While you can use ConfigMaps; ConfigMaps are untyped and sometimes using a more strongly typed API extension object provides clarity and ease of use. Extensions that are just data don't need a corresponding controller to activate them, but they may have validating or mutating admission controllers to ensure that they are well formed.

### Compilers

This pattern knows as "compiler" or "abstraction" pattern has the API extension object representing a higher-level abstraction that is "compiled" into a combination of lower level Kubernetes objects. The LoadTest extension we discussed prior is an example of this pattern. A user consumes the extension as a high level concept, in this example a `loadtest`, but it comes into being by being deployed as a collection of Kubernetes Pods and services. To achieve this, a compiled abstraction requires an API controller to be running somewhere in the cluster, to watch the current LoadTests and create the "compiled" representation. There is however no online health maintenances.

### Operators

Operator pattern provides online proactive management of the resources created by the extension. These extensions likely provide a higher level abstraction (for example a database) that is compiled down to a lower level representation, but they also provide online functionality such as snapshot, backups, or upgrades. To achieve this, the controller not only monitors the running state of the application supplied by the extension to add or remove things as necessary, but also monitors the running state of the application supplied by the extension and takes actions to remediate unhealthy databases, take snapshots or restore from a snapshot if failure occurs. Operators are the most complicated pattern for API extension.

## Getting started

The [Kubebuilder project](https://kubebuilder.io) is a good resource to use when getting started with extension development.
