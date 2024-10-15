---
layout: page
title: "Kubernetes ReplicaSets"
permalink: /kubernetes/replica_sets
---

[comment]: <> (TODO: Need to organize and review this section to remove redundancy and make it better for reference use.)

## What are ReplicaSets

In real world use cases you rarely run a single Pod. Typically multiple pods are run for redundancy, scale, sharding or other reasons. A replicated set of Pods should be a single entity (to prevent from repetitive work). The **ReplicaSet** object in Kubernetes acts as a cluster wide Pod manager, ensuring that the right types and numbers of Pods are running at all times. ReplicaSets not Pods tend to be the building blocks of applications in Kubernetes. ReplicaSets provide self-healing for our applications at the infrastructure level. Pods managed by ReplicaSets are automatically rescheduled under certain failure conditions such as node failures and network partitions.

## Adopting exiting containers

Though ReplicaSets create and manage Pods, they do not own the Pods they create. ReplicaSets use label queries to identify the set of Pods they should be managing. They then use the general Kubernetes APIs to create the Pods they are managing. This decoupling of Pods and ReplicaSets supports several important behaviors. Because ReplicaSets are decoupled from the Pods they manage, you can create a ReplicaSet that will "adopt" an existing Pod and scale out additional copies of those containers. This is a path to go from a single imperatively declared Pod to a replicated set of Pods managed by ReplicaSet.

## Quarantining Containers

When a server misbehaves, Pod-level health checks will automatically restart the Pod. If your health checks are incomplete, a Pod can be misbehaving, but still be a part of the replicated set. In this scenario you can kill the Pod, but then developers would only have logs to troubleshoot with. A better option is to leave the Pod running but quarantine it so that it is not sent traffic. You can do this modifying the set of labels on the Pod. This will disassociate it from the ReplicaSet, but leave it running for developers to troubleshoot.

## Designing with ReplicaSets

ReplicaSets are designed to represent a single, scalable microservice inside your architecture. The key characteristic of ReplicaSets is that every Pod that is created by the ReplicaSet controller is entirely homogenous. Typically, these Pods are then fronted by a Kubernetes service load balancer, which spreads traffic across the Pods that make up the service. Generally speaking ReplicaSets are designed for stateless (or nearly stateless) services. When it is scaled down an arbitrary Pod wil be selected for deletions and this should not impact your application.

## ReplicaSet spec

All ReplicaSets must have a unique name (defined using the `metadata.name` field), a `spec` section that describe the number of Pods (replicas) that should be running in the cluster, and a Pod template that describes the Pod to be created.

Below is an example of a minimal ReplicaSet

[comment]: <> (TODO: Try this and replace with valid code.)

```yaml
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: kuard
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: kuard
        version: "2"
    spec:
      containers:
        - name: kuard
          image: "gcr.io/kuar-demo/kuard-amd64:green"
```

## Inspecting a ReplicaSet

You can use the command `kubectl describe rs <ReplicaSet name>` to get details about a running ReplicaSet

## Finding a ReplicaSet from a Pod

The ReplicaSet controller adds an annotation to every Pod that it creates. The key for the annotation is `kubernetes.io/created-by` If you run the command `kubectl get pods <pod-name> -o yaml` it will provide the ReplicaSet that is managing the Pod. Note that these annotations are best effort and while added when the Pod is created could be removed by a Kubernetes user at any time.

## Finding a set of Pods for a ReplicaSet

You can determine the Pods managed by a ReplicaSet by using a label query. You can use `kubectl describe` to find the set of labels from the ReplicaSet. Then do a label query such as `kubectl get pods -l app=kuard,version=2` to find the Pods. This is the same query that ReplicaSet executes to determine the current numbers of Pods.

## Scaling ReplicaSets

Replica sets are scaled up or down by updating the `spec.replicas` key on the ReplicaSet object stored in Kubernetes. When a ReplicaSet is scaled up, new Pods are submitted to the Kubernetes API using the Pod template defined on the ReplicaSet.

### Imperative scaling ReplicaSets with kubectl

`kubectl scale replicasets <ReplicaSet name> --replicas<number>` for example `kubectl scale replicasets kuard --replicas=4`

This works, and can be useful for demos or troubleshooting, but in typical operations declarative approach should be used.

### Declaratively scaling ReplicaSets with kubectl apply

The declarative approach is to make changes in a version controlled file and then applying those changes to our cluster with the command `kubectl apply -f <manifest_file.yaml>` for example `kubectl apply -f kuard-rs.yaml`

### Autoscaling a ReplicaSet

Kubernetes supports scaling ReplicaSets using **HPA (Horizontal Pod Autoscaling)**. HPA requires the presence of a `heapster` Pod on your cluster which keeps track of metrics and provides an API for consuming those metrics that HPA uses to make decisions. Most Kubernetes installations include `heapster` by default.

#### Side note on scaling

Kubernetes does not currently support vertical scaling (adding CPU for example) but this is planned. Many solution offer cluster auto scaling where extra nodes are added as needed.

#### Autoscaling based on CPU

Scaling based on CPU is the most common use case for Pod autoscaling. Its most useful for request-based systems that consume CPU proportionally to the number of requests they are receiving while using a relatively static amount of memory.

Use the command `kubectl autoscale rs kuard --min=2 --max=5 --cpu-percent=80`

To view, modify or delete this resource you can use the standard kubectl commands and the `horizontalpodautoscalers` resource (you can shorten that to `hpa`) for example `kubectl get hpa`.

Because of the decoupled nature of Kubernetes, there is no direct link between HPA and the ReplicaSet. This leads to some anti patterns. Its a bad idea to combine both autoscaling and imperative or declarative management of the number of replicas. If both you and an autoscaler are attempting to modify the number of replicas, it's highly likely that you will clash, resulting in unexpected behavior.

## Deleting ReplicaSets

When a ReplicaSet is no longer required, it can be deleted using the `kubectl delete` command. For examples `kubectl delete rs kuard`. By default this also deletes the Pods that are managed by the ReplicaSet. If you don't want to delete the Pods use the command similar to `kubectl delete rs kuard --cascade=false`.
