---
layout: page
title: "Kubernetes Pods"
permalink: /kubernetes/pods
---

[comment]: <> (TODO: Break down this section once you have more/complete content)

## What is a Pod in Kubernetes?

Kubernetes groups multiple containers into a single atomic unit called a **Pod**. A Pod represents a collection of application containers and volumes running in the same execution environment. Pods (not containers) are the smallest deployable artifact in a Kubernetes cluster. All containers in a pod land on the same machine. Each container within a Pod runs in its own cgroup, but they share a number of Linux namespaces. Applications running in the same Pod share the same IP address and port space (network namespace), have the same hostname (Unix Time Sharing (UTS) namespace) and can communicate using native inter-process communication channels over System V IPC or POSIX message queues (IPC namespace). Applications in different pods are isolated from each other. They have different IP addresses, different host names, and more. Containers in different Pods running on the same node may as well be on different servers.

[comment]: <> (TODO: Paragraph above has Linux concepts that I should link to my Linux notes. cgroup, namespace and POSIX)

When designing Pods ask yourself if these containers will work the same if they are on different machines. If the answer is no Pod is the correct grouping for the containers. If the answer is yes then multiple Pods is probably the correct answer.

## Pod Manifests

Pods are described in a Pod manifest. The Pod manifest is just a text file representation of the Kubernetes API object. The Kubernetes API server accepts and processes Pod manifests before storing them in persistent storage (etcd). The scheduler also uses the Kubernetes API to find Pods that haven't been scheduled to a node. The scheduler then places the Pods onto nodes depending on the resources and other constraints expressed in the Pod manifests. Kubernetes tries to ensure that Pods from the same application are distributed onto different machines for reliability. Once scheduled to a node, Pods don't move and must be explicitly destroyed and rescheduled or if a node fails. To create multiple pods you want to use ReplicaSets.

[comment]: <> (TODO: Make links to the Kubernetes concepts in paragraph above.)

Below is an example of a Pod manifest file.

[comment]: <> (TODO: Example below has a lot of concepts not covered in text above it. Should move it down or replace it with a more basic one.)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  volumes:
    - name: "kuard-data"
      nfs:
        server: my.nfs.server.local
        path: "/exports"
  containers:
    - image: gcr.io/kuar-demo/kuard-arm64:blue
      name: kuard
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
        limits:
          cpu: "1000m"
          memory: "256Mi"
      volumeMounts:
        - mountPath: "/data"
          name: "kuard-data"
      livenessProbe:
        httpGet:
          path: /healthy
          port: 8080
        initialDelaySeconds: 5
        timeoutSeconds: 1
        periodSeconds: 10
        failureThreshold: 3
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 30
        timeoutSeconds: 1
```

## Pod Resource Limits

Kubernetes tracks both the **requests** and **upper limits** for resources for each Pod that runs on a machine. Resources requested by a Pod are guaranteed to be present on the node, while a Pod's limit is the maximum amount of a given resource that a Pod can consume. A Pod's limit can be higher than its request, in which case the extra resources are supplied on a best effort basis. They are not guaranteed to be present on a node.

### Resource management

Kubernetes allows users to specify two different resource metrics. Resource **requests** specify the minimum amount of a resource required to run the application. Resource **limits** specify the maximum amount of a resource that an application can consume.

#### Resource requests: minimum required resources

A Pod requests resources required to run its containers. Kubernetes guarantees that these resources are available to the Pod. Most commonly CPU and memory is requested, but Kubernetes supports others such as GPUs.

Resources are requested per container not per Pod. The total resources requested by the Pod is the sum of all resources requested by the containers in the Pod. The reason for this is containers may have different CPU requirements.

#### Request limit details

The scheduler makes sure that the resource requests are available and do not exceed the capacity of the node. The request specifies a minimum, but not the maximum a Pod can use. A Pod can go above its requested limits as long as they are available on the system. CPU requests are handled using `cpu-shares` functionality in the Linux kernel.  For memory since the OS cannot just remove memory from a process once its allocated so when system runs out of memory `kubelet` terminates containers whose memory usage is greater thant heir requested memory. These container are restarted but with less available memory for the container to use.  Limits can be used to place an upper bound and not let a containers use unlimited resources.

## Health Checks

[comment]: <> (TODO: Should revisit and decide if Helth and Readiness Check below with Pods or someplace else)

If Kubernetes detects that the main process of your application is not running it will restart it. In most cases a simple process check is insufficient to verify health (process may be running but deadlocked). To address this Kubernetes introduced health checks for application *liveness*. Since these health checks are application specific you have to define them in your Pod manifest (see example above). In addition to HTTP checks, Kubernetes supports tcpSocket health checks that open a TCP socket; if the connection is successful, the probe succeeds. Kubernetes also allows `exec` probes. These execute a script or program in the context of the container. If this script returns a zero exit code, the probe succeeds; otherwise it fails.

Kubernetes also provides a *readiness* probe. Kubernetes makes a distinction between liveness and readiness. Liveness determines if an application is running properly. Containers that fail liveness are restarted. Readiness describes when a container is ready to serve user requests. Containers that fail readiness checks are removed from load balancers. Readiness probes are configured similarly to liveness probes. Combining readiness and liveness probes helps ensure only healthy containers are running within the cluster.