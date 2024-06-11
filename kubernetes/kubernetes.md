---
layout: page
title: "Kubernetes"
permalink: /kubernetes
---

[comment]: <> (TODO: Should put the Docker stuff into its own docker section)

[comment]: <> (TODO: Need to organize this monolith)

## Docker images

There are some gotchas to keep in mind about Docker images.  

* Files that are removed by subsequent layers in the system are actually still present in the images; they are just inaccessible.  For example if layer A has a large file named BigFile and layer B removed the BigFile, layer C which builds on will still have the BigFile and that will have impact every time you need to publish or pull the image C even though the file is inaccessible.

* Each layer is an independent delta from the layer below it. Every time you change a layer, it changes every layer that comes after it. Changing the preceding layers means that they need to be rebuilt, re-pushed, and re-pulled to deploy your image to development. For example if you have a layer with source code and a layer that adds the needed libraries you want the source code layer which is more likely to change to be the subsequent layer.  In general you want to order your layers from least likely to change to most likely to change to optimize the image size for pushing and pulling.

* Secrets and images should never be mixed. Never put secrets into your images.

### Multistage image builds

One of the most common ways to unintentionally build large images is to do the actual program compilation as part of the construction of the application container image. This may leave development tools and artifacts in your image which can be quite large.  To resolve this issue Docker intruduced **multistage builds**. With multi stage builds instead of producing a single image a Docker file can produce multiple images with each image considered a stage. Artifacts can be copied from preceding stages to the current stage.

[comment]: <> (TODO: Need to try this and write up a very basic example  The book uses kuard as example but that is too comlex for a reference.)

## Pods

Kubernetes groups multiple containers into a single atomic unit called a *Pod*. A Pod represents a collection of application containers and volumes running in the same execution environment. Pods not containers are the smallest deployable artifact in a Kubernetes cluster. All containers in a pod land on the same machine. Each container within a Pod runs in its own cgroup, but they share a number of Linux namespaces. Applications running int he same Pod share the same IP address and port space (network namespace), have the same hostname (UTS namespace) and can communicate using native inter-process communication channels over System V IPC or POSIX message queues (IPC namespace).  Applications in different pods are isolated from each other. They have different IP addresses, different host names, and more. Containers in different Pods running on the same node may as well be on different servers.

When designing Pods ask yourself if these containers will work the same if they are on different machines. If the answer is no Pod is the correct grouping for the containers. If the answer is yes then multiple Pods is probably the correct answer.

Groups of containers, can group together images developed by different teams into a single deployable unit.

Pods are described in a Pod manifest. The Pod manifest is just a text file representation of the Kubernetes API object. The Kubernetes API server accepts and processes Pod manifests before storing them in persistent storage (etcd). The scheduler also uses the Kubernetes API to find Pods that haven't been scheduled to a node. The scheduler then places the Pods onto nodes depending on the resources and other constraints expressed in the Pod manifests. Kubernetes tries to ensure that Pods from the same application are distributed onto different machines for reliability. Once scheduled to a node, Pods don't move and must be explicitly destroyed and rescheduled or a node fails.  To create multiple pods you want to use ReplicaSets.

Below is an example of a Pod manifest file.

[comment]: <> (TODO: Example below has a lot of concepts not covered in text above it. Should move it down.)

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

Kubernetes tracks both the *requests* and *upper limits* for resources for each Pod that runs on a machine. Resources requested by a Pod are guaranteed to be present on the node, while a Pod's limit is the maximum amount of a given resource that a Pod can consume. A Pod's limit can be higher than its request, in which case the extra resources are supplied on a best effort basis. They are not guaranteed to be present on a node.

### Health Checks

If Kubernetes detects that the main process of your application is not running it will restart it. In most cases a simple process check is insufficient to verify health (process may be running but deadlocked). To address this Kubernetes introduced health checks for application *liveness*. Since these health checks are application specific you have to define them in your Pod manifest (see example above).  In addition to HTTP checks, Kubernetes supports tcpSocket health checks that open a TCP socket; if the connection is successful, the probe succeeds.  Kubernetes also allows `exec` probes. These execute a script or program int he context of the container.  If this script returns a zero exit code, the probe succeeds; otherwise it fails.

Kubernetes also provides a *readiness* probe. Kubernetes makes a distinction between liveness and readiness.  Liveness determines if an application is running properly. Containers that fail liveness are restarted. Readiness describes when a container is ready to serve user requests. Containers that fail readiness checks are removed from load balancers. Readiness probes are configured similarly to liveness probes.  Combining readiness and liveness probes helps ensure only healthy containers are running within the cluster.

### Resource management

Kubernetes allows users to specify two different resource metrics. Resource **requests** specify the minimum amount of a resource required to run the application. Resource **limits** specify the maximum amount of a resource that an application can consume.

#### Resource requests: minimum required resources

A Pod requests resources required to run its containers. Kubernetes guarantees that these resources are available to the Pod. Most commonly CPU and memory is requested, but Kubernetes supports others such as GPUs.

Resources are requested per container not per Pod. The total resources requested by the Pod is the sum of all resources requested by the containers in the Pod. The reason for this is containers may have different CPU requirements.

#### Request limit details

The scheduler makes sure that the resource requests are available and do not exceed the capacity of the node. The request specifies a minimum, but not the maximum a Pod can use. A Pod can go above its requested limits as long as they are available on the system. CPU requests are handled using `cpu-shares` functionality in the Linux kernel.  For memory since the OS cannot just remove memory from a process once its allocated so when system runs out of memory `kubelet` terminates containers whose memory usage is greater thant heir requested memory. These container are restarted but with less available memory for the container to use.  Limits can be used to place an upper bound and not let a containers use unlimited resources.

### Persisting Data with Volumes

#### Using volumes with pods

To add a volume to a Pod manifest you need to add the `spec.volumes` section which defines all the volumes that may be accessed by containers in the Pod manifest. Not all containers are required to mount all of the volumes int the Pod. You also need to add `volumeMounts` array in the container definition which defines volumes mounted into a particular container and the path they are mounted at (there are examples of both in the sample manifest). Two different containers in a Pod can mount the same volume at different mount paths.

Below are some examples of how volumes can be used in Kubernetes

#### Communication/synchronization

Have two containers mount same file system to use it as a communication/synchronization mechanism. One container can periodiclaly pull down latest code from Git and another container can serve that code content for example.

#### Cache

Volume can be used as a cache while container is running. The volume will survive a restart so the cached content will persist till contaienr is deleted.

#### Persistent Data

Sometimes a volume will be used for truly persistent data the life span of which is independent of the container using it. The volume should also be able to move from host to host. Kubernetes supports a bunch of options for this including NFS, iSCSI, EBS, Azure Filet Disk, etc.

#### Mounting the host filesystem

If you don't need a persistent volume but do need access to underlying file system for example access to /dev for devices. For these scenarios Kubernetes supports `hostPath` volumes which can mount arbitrary locations on the worker node into the container.

#### Persisting Data Using Remote Disks

Kubernetes supports NFS and iSCSI as mentioned, but cloud providers have support for this as well, and will often create the volume for you if one des not exist.

## Namespaces

Provide isolation and access control so that each microservice can control the degree to which other services interact with it. Namespaces can be used to isolate developer environments. For example instead of setting up a cluster for each developer you can have one cluster and have a namespace for each developer. This makes it conceivable to deploy and test every single commit. 

## Ingress

Objects provide an easy to use frontend that can combine multiple micro-services into a single external API surface area.

## Nodes

Nodes are divided into **master** nodes and **worker** nodes. The master node will run containers such as the API server scheduler, etc. which manage the cluster. Kubernetes does not generally schedule work onto master nodes to ensure user workloads don't harm the overall operation of the cluster.

## System containers vs application containers

System containers seek to mimic virtual machines and often run a full boot process. They often include a set of system services typically found on a VM, such as ssh, cron and syslog. Over time this approach became seen as poor practice and application containers gained favor. Application containers commonly run a single problem. While a single program per container may seem like an unnecessary constrains, it provides the perfect level of granularity for composing scalable applications and is a design philosophy heavily used by Pods.

## Cluster Components

Many of the components that make up Kubernetes are deployed using Kubernetes itself.  Some of these components are...

### Kubernetes proxy

The kubernetes proxy is responsible for routing network traffic to load-balanced services in the Kubernetes cluster. To do this job the, the proxy must be present on every node int he cluster.

### The Docker Container Runtime

Kubernetes relies on a container runtime to actually run containers.  The interface to this container runtime is defined by the **Container Runtime Interface (CRI)** standard.  The CRI API is implemented by a number of different programs, including `containerd-cri` built by Docker and the `cir-o` implementation from Red Hat.

### Kubelet

Kubelet is the daemon on each node that launches containers. Under the hood Kubernetes uses `cgroup` technology to restrict resource utilization.

### Controller-manager

The `controller-manager` is responsible for running various controllers that regulate behavior in the cluster; for example, ensuring that tall of the replicas of a service are available and healthy.

### scheduler

The `sceduler` is responsible for placing different pods onto different nodes in the cluster.

### etcd

`etcd` server is the storage for the cluster where all of the API objects are stored.

### Kubernetes DNS

Kubernetes runs a DNS server, which provides naming and discovery for the services that are defined in the cluster. This DNS server also runs as a replicated service on the cluster. Depending on the size of your cluster, you may see one or more DNS servers running in your cluster.

### Kubernetes UI

The UI is run as a single replica, but it is still managed by a Kubernetes deployment for reliability and upgrades. You can use kubectl proxy to access this UI.  If you run the command `kubectl proxy` you should be able to get tot he UI at `http://localhost: 8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/`.  

This component may not always be installed. 

## Kubectl

`kubectl` is the official Kubernetes client. Its a command line tool for interacting with Kubernetes API. It can be used to manage most Kubernetes objects as well as explore and verity the overall health of the cluster.

### Checking cluster status

`kubectl version` will display the version of the kubectl tool as well as the version of the server.

`kubectl get componentstatuses` will give you a quick diagnostic for the cluster.

### Listing Kubernetes worker nodes

`kubectl get nodes` lists nodes

`kubectl describe nodes node-1` Give you details about a node hardware, resource utilizations and pods its running.

### Working with namespaces

By default, kubectl tool interacts with the `default` namespace. If you want to use a different namespace, you can pass the `--namespace` flag for example `kubectl --namespace=mystuff`

If you want to change the default namespace more permanently you can use a **context**. This gets recorded in a kubectl file usually located ate `$HOME/.kube/config`.  To create a context with a different default you would run the command `kubectl config set-context my-context --namespace=mystuff`. This will create a new context but does not actually start using it. To use this context you run the command `kubectl config use-context my-context`. Contexts can be used to manage different clusters or different users for authenticating to those clusters using the `--users` or `--clusters` flags with the `set-context` command.

### kubectl display formats

You can use the `-o wide` modifier to get more details (in less human readable format that Kubernetes tries to limit to a single line) when using `get` to display resources. You can also use `-o json` to get a JSON output which will have full details. Another common task is extracting specific field from the object.  `kubectl` uses the JSONPATH query language to select fields int eh returned object. For example

`kubectl get pods my-pod -o jsonpath --template={.status.podIP}`

### kubectl describe

If you are interested in details for an object use the command `kubectl describe <resource-name> <obj-name>`. This will provide a rich multi line human readable description of the object as well as related objects and events.

### Creating, updating, and destroying Kubernetes objects

Objects in Kubernetes are represented as JSON or YAML files. You can use these YAML or JSON files to create, update or delete object on the Kubernetes server.

If for example you have a simple object stored in `obj.yaml`; you can use `kubectl` to create this object by running the command `kubectl apply -f obj.yaml`. All the details for the object being created will be obtained from the file.  if you make changes to the file, you can use the same command to apply those changes. The `apply` tool will only modify objects that are different from teh current objects in the cluster. If an object already exists it will simply exit successfully without making any changes.

If you want to see what the `apply` command will do without making the changes you can use the `--dry-run` flag to print the objects to the terminal without sending them to the server.

If you want to make iterative changes instead of editing a local file you can use the command `kubectl edit <resource-name> <obj-name>` which will download the latest object state and launch an editor for you. After you save the file, it will be automatially uploaded back tot he Kubernetes cluster.

The `apply` command also records the history of previous configurations in an annotation. You can manipulate these records with the `eit-last-applied`, `set-last-applied` and `view-last-applied` commands.  For example `kubectl apply -f myobj.yaml view-last-applied` will show you the last state that was applied to the object. The `-f` option is to specify the file name.

When you want to delete an object use the command `kubectl delete -f obj.yaml`. ***Note:*** `kubectl` will not prompt for a confirmation before deleting an object.  There is however a terminating grace period of 30 seconds by default (At least for Pods need to confirm if for other objects). When a Pod is transitioned to Terminating state it stops taking request.  The grace period allows the Pod to finish any active reqeusts that it may be in the middle of processing before it is terminated. When a Pod is deleted any data in the container is deleted as well.  If you need to persist data than you need to use `PersistentVolumes`

To create a deployment in a non declarative way you can use the command `kubectl create deployment <deployment_name> --image=<image> --replicas=<replica_count>` with real values that looks like `kubectl create deployment alpaca-prod --image=gcr.io/kuar-demo/kuard-arm64:blue --replicas=2`

## Kubernetes API

Everything contained in Kubernetes is represented by a RESTful resource. These objects exist at unique HTTP paths; for example, `https://your-k8s.com/api/v1/namespaces/default/pods/my-pod` leads to the representation of a Pod in the default namespace. The `kubectl` command makes HTTP requests to these URLs to access Kubernetes objects that reside at these paths.

## Labeling and annotating objects

Labels and annotations are tags for your objects. You can update the labels and annotation on any Kubernetes object using the `annotate` and `label` commands. The syntax for both is identical. To add a label use the command `kubectl label pods bar color=red`. By default `label` and `annotate` will not let you overwrite an existing label. To remove a label you would use the command `kubectl label pods bar color-`

## Debugging Commands

To see the logs for a running container use `kubectl logs <pod-name>`.  If you have multiple containers in your Pod you can specify the container with the `-c` flag.  You can also use the `-f` flag to stream the logs.

To execute commands in a running container use the command `kubectl exec -it <pod-name> -- bash`. This will provide you with an interactive shell inside the running container so that you can perform more debugging.  If you don't have bash or another terminal you can always attach to the running process using `kubectl attach -it <pod-name>`. This will attach to the running process and allow you to send input to the running process (assuming process is set up to read from stdin)

You can copy files to and from a container using the cp command: `kubectl cp <pod-name>:</path/to/remote/file> </path/to/local/file>`. Generally speaking copying files to a pod is an anti patterns, but it may be useful when putting out a fire.

If you want to access your Pod via the network, you can use the `port-forward` commands to forward network traffic from the local machine tot he Pod. This enabled you to securely tunnel network traffic through to containers that might not be exposed anywhere on the public network. For example the following command opens up a connection that forward traffic from the local machine on port 8080 to the remote container on port 80.

`kubectl port-forward <pod-name> 8080:80`

You can also use the `port-forward` command with services by specifying `services/<service-name>` instead of `<pod-name>`, but note that the requests for a service will be forwarded to a single Pod, not go though the service load balancer.

If you are interested in how your cluster is using resources you can use `kubectl top nodes` or `kubectl top pods` to see the pods running and their resource utilization.

## Tools and resources

* The [kind](https://kind.sigs.k8s.io) project lets you run Kubernetes using containers as nodes so you can play around in your local environment.

* There are plugins for VSCode, IntelliJ, Eclipse etc for working with your cluster.

* `kubectl` supports integration with your shell to enable tab completion for both commands and resources. You may need to install the `bash-completion` package to make this work.

* For log aggregators look into `fluentd` and `elasticsearch`.

## Labels and Annotations

*Labels* are key value pairs that can be attached to Kubernetes objects such as Pods and ReplicaSets. They are useful for attaching identifying information to Kubernetes objects. Labels provide the foundation for grouping objects. *Annotations*, on the other hand, provide a storage mechanism that resembles labels. Annotations are key value pairs designed to hold non-identifying information that can be leveraged by tools and libraries.

### Labels

Labels are key/value pairs where both the key and value are represented by strings. Label keys can be broken down into two parts: an optional prefix and a name, separated by a slash. The prefix, if specified must be a DNS sub-domain with a 253 character limit. The key name is required and must be shorted than 63 characters. Names must also start and end with an alphanumeric character and permit the use of dashes `-`, underscores`_` and dots `.`. Label values are strings with a maximum length of 63 characters. The contents of the label values follow the same rules as for label keys. Below are some examples of label keys and values

| Key | Value |
| --- | ----- |
| acme.com/app-version | 1.0.0 |
| appVersion | 1.0.0 |
| app.version | 1.0.0 |
| kubernetes.io/cluster-service | true |

When domain names are use in labels and annotations they are expected to be aligned to that particular entity in some way. For example a project might define a canonical set of labels used to identify the various stages of application deployment (e.g staging, canary, production)

#### Applying and removing labels

To apply or update a label on a running object use the command similar to `kubectl label deployments alpaca-prod "ver=1"` ***Note:*** this command only labels the deployment not the objects which the deployment creates. To change those objects you would need to change the template embedded in the deployment.

You can view labels on an object using the `kubectl describe` command.  You can also use the `--show-labels` option in kubectl to see the labels applied to an object. For example `kubectl get pods --show-labels`

To remove a label use a minus at end of label command specifying a key for example `kubectl label deployments alpaca-prod "env-"`. The minus `-` at end of label name removes it.

You may see a label called `pod-template-hash` when viewing labels. This label is applied by the deployment so it can keep track of which Pods were generated from which template version and allows the deployment to manage updates ina clean way.

#### Label Selectors

Label selectors are used to filter Kubernetes objects based on a set of labels. Label selectors are used by both end users and by different types of objects (for example how a ReplicaSet relates to its Pods).

You use the `--selector` flag or `-l` for short to use label selectors in kubectl. Below are a few examples.

List pods with `ver` label set to 2 `kubectl get pods --selector="ver=2"`

If you specify two selectors separated by a comma only objects that satisfy both will be returned. This is a logical AND operation: `kubectl get pods -l="app=badnicoot,ver-2"`

You can also ask if a label is one of a set of values. `kubectl get pods -l="app in (alpaca,bandicoot)"`

We can also check if a label is set or not. `kubectl get deployments -l="canary"`. This example we are asking for all deployments where the `canary` label is set to anything.

Below is a summary of the Selector operations:
| Operator | Description |
| -------- | ----------- |
| key=value | key is set to value |
| key!=value | key is not set to value |
| key in (value1, value2) | key is one of value1 or value2 |
| key notin (value1, value2) | key is not one of value1 or value2 |
| key | key is set |
| !key | key is not set |

You can combine positive and negative selectors for example `kubectl get pods -l "ver=2,!canary"`

#### Labels in Kubernetes architecture

Kubernetes is a purposefully decoupled system. There is no hierarchy and all components operate independently. When objects need to be related to each other they are related with labels.  For example ReplicaSets which create multiple replicas of a Pod, find the Pods that they are managing via a selector. When you want to restrict network traffic in your cluster, you would use networkPolicy in conjunction with specific labels to identify Pods that should or should not be allowed to communicate with each other. Labels are the glue that holds a Kubernetes application together.

### Annotations
