---
layout: page
title: "Kubernetes Cluster Components"
permalink: /kubernetes/cluster_components
---

Many of the components that make up Kubernetes are deployed using Kubernetes itself. Some of these components are covered below.

## Kubernetes proxy

The kubernetes proxy is responsible for routing network traffic to load-balanced services in the Kubernetes cluster. To do this job the, the proxy must be present on every node int he cluster.

## The Docker Container Runtime

Kubernetes relies on a container runtime to actually run containers. The interface to this container runtime is defined by the **Container Runtime Interface (CRI)** standard. The CRI API is implemented by a number of different programs, including `containerd-cri` built by Docker and the `cir-o` implementation from Red Hat.

## Kubelet

Kubelet is the daemon on each node that launches containers. Under the hood Kubernetes uses `cgroup` technology to restrict resource utilization.

## Controller-manager

The `controller-manager` is responsible for running various controllers that regulate behavior in the cluster; for example, ensuring that tall of the replicas of a service are available and healthy.

## scheduler

The `scheduler` is responsible for placing different pods onto different nodes in the cluster.

## etcd

`etcd` server is the storage for the cluster where all of the API objects are stored.

## Kubernetes DNS

Kubernetes runs a DNS server, which provides naming and discovery for the services that are defined in the cluster. This DNS server also runs as a replicated service on the cluster. Depending on the size of your cluster, you may see one or more DNS servers running in your cluster.

## Kubernetes UI

The UI is run as a single replica, but it is still managed by a Kubernetes deployment for reliability and upgrades. You can use kubectl proxy to access this UI.  If you run the command `kubectl proxy` you should be able to get tot he UI at `http://localhost: 8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/`.  

This component may not always be installed.

## [Kubectl]({% link kubernetes/kubectl.md %}) - The official Kubernetes client

## Kubernetes API

Everything contained in Kubernetes is represented by a RESTful resource. These objects exist at unique HTTP paths; for example, `https://your-k8s.com/api/v1/namespaces/default/pods/my-pod` leads to the representation of a Pod in the default namespace. The `kubectl` command makes HTTP requests to these URLs to access Kubernetes objects that reside at these paths.
