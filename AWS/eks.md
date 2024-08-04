---
layout: page
title: "AWS EKS"
permalink: /aws/eks
---

## Random notes from doc review

The control plane positions at least two API server instances and three etcd instances across three AWS Availability Zones within an AWS Region. If a control plane instance falters, Amazon EKS quickly replaces it using different Availability Zone if needed.

EKS Cluster components can't view or receive communication from other clusters or AWS accounts, except when authorized by Kubernetes role based access control (RBAC) policies.

Kubernetes control plane components

* API server (kube-apiserver) - exposes the Kubernetes API so requests to the cluster can be made from both inside and outside of the cluster. For example both kubectl and kubelet talk to the API server one to issue commands adn the other to check for status of a pod.

* etcd key value store - etcd is used to keep track of the current state of the cluster. If `etcd` service becomes unavailable, you would be unable to update or query the status of the cluster (though workloads can run for a while). Critical clusters typically have multiple load-balanced instances of the `etcd` service running and do periodic backups `etcd` key value contents.

* Scheduler - Scheduler determines which node or nodes for replicas should run a pod. If resources to run a pod are not available the request will fail unless you made other provisions such as Managed Node Groups or Karpenter that can automatically start up new nodes to handle the workloads.

* Controller Manager - The Controller Manger runs a daemon process (kube-controller-manager) to watch the state of the cluster and make changes to the cluster to re-establish the expected states. There are several controllers that watch over different Kubernetes objects which include `node-lifecycle-controller`, `statefulset-controller`, `endpoint-controller`, `cronjob-controller` as well as others.

* Cloud Controller Manager - Interactions between Kubernetes and the cloud provider that carries out requests for the underlying data center resources are handled by the Cloud Controller manger. Controllers managed by the Cloud Controller Manager can include a route controller (for setting up network routes), service controller (for using cloud load balancing services), and node controller (for using cloud APIs to keep Kubernetes nodes in sync with cloud nodes)

Worker nodes are what we call the data plan.  Components in the data plane are as follows

* kubelet - Each node runs kubelet which is what the API server communicates with to make sure node is properly registered and pods requested by the scheduler are running. Kubelet can read the Pod manifests and set up storage volumes or other features needed by the Pods on the local system. It can also check on health of locally running containers.

* container runtime - The Container Runtime is responsible for pulling container images from appropriate registry, start/stop the container and respond to queries about the container. The default as of Kubernetes 1.24 is `containerd` You can optionally install Docker if you want.

* kube-proxy - kube-proxy is responsible for communication between Pods using Services.  Kubernetes needs a way to set up Pod networks to track IP addresses and ports associated with those Pods. The kube-proxy service runs on every node to allow that communication between Pods to take place.

A Kubernetes workload is just an application running on Kubernetes. This can be containers running in Pods or a batch job.

Besides docker other tools that you an use the build container are `podman` and `nerdctl`.

A Persistent Volume will stay after Pod is deleted, while an Ephemeral Volume is deleted when the Pod is deleted. A cluster admin can create different StorageClasses for your cluster which you can choose via attributes.

Pods can be disrupted involuntarily (a node goes down) or voluntarily (an update is desired). You can configure a Pod dsruption budget to have some control over how available your application remains when disruptions occur.

The ability for traffic that originates somewhere else to reach your nodes is called **ingress**. Traffic that originates from the nodes and leaves the network is called **egress**.

When creating a VPC there are 3 sets of subnets you should consider creating.

1. Subnet for servers that will receive traffic from clients on the internet
2. Subnet for servers that will receive traffic only from other servers in the VPC
3. Subnet for servers that will receive traffic only though a VPN connection to your network.

If your application will receive traffic from the internet, the VPC must have an internet gateway. Attaching an internet gateway to a VPC does not make your instnaces accessible from the internet. In addition to attaching the internet gateway you must update the subnet route toble with a route to the internet gateway. You must also ensure that the instances have public IP addresses and an associated security group that allows traffic from the internet over specific ports and protocols required by your applications.

If you want to talk to outside world for patches, but not have inbound traffic you need to run a NAT gateway.

The allowed block size for a VPC CIDR block is between a /16 netmask (65,536 IP addresses) or a /25 (16 IP addresses). 

While RFC1918 reserves the below for private networks avoid using the 172.16 one as that is used by some AWS services such as sage maker and you may get IP conflicts.

```text
10.0.0.0        -   10.255.255.255  (10/8 prefix)
172.16.0.0      -   172.31.255.255  (172.16/12 prefix)
192.168.0.0     -   192.168.255.255 (192.168/16 prefix)
```