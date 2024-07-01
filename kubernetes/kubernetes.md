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

To create a deployment in a non declarative way you can use the command `kubectl create deployment <deployment_name> --image=<image> --replicas=<replica_count>` with real values that looks like `kubectl create deployment alpaca-prod --image=gcr.io/kuar-demo/kuard-arm64:blue --replicas=3 --port=8080`

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

Annotations provide a place to store additional metadata for Kubernetes object with the sole purpose of assisting tools and libraries.  They are a way for other programs driving Kubernetes via an API to store some data with an object. Annotations can be used for the tool itself, or to pass configuration information between external systems. While labels are used to identify and group objects, annotation are used to provide extra information about where an object came from, how to use it, or policy around that object. 

When in doubt use an annotation and promote it to a label if there is need to use it as a selector.

Annotations are used to...

* Keep track of a "reason" for the latest update to an object
* Communicate a specialized scheduling policy to a specialized scheduler
* Extend data bout the last tool to update the resource and how it was updated (detect changes and do smart merge)
* Attach build, release, or image information that isn't appropriate for labels (Git hash, timestamp PR number etc)
* Enable the Deployment object to keep track of ReplicaSets that it is managing for rollouts
* Prototype alpha functionality in Kubernetes (instead of creating first class API filed, the parameters for that functionality are encoded in an annotation)

Annotations are used in various places in Kubernetes with the primary use case being rolling deployments. During rolling deployments, annotations are used to track rollout status and provide the necessary information required to roll back a deployment to a previous state.

#### Defining annotations

Annotations keys use the same format as label keys but because they are often used to communicate information between tools the namespace part of the key is more important.  Example keys would be `deployment.kubernetes.io/revision` or `kubernetes.io/change-cause`.

The value component of an annotation is a free-form field and there is no validation that any format is being followed. Its not uncommon to have a JSON document encoded as a string and stored as an annotation.

Annotations are defined in the common `metadata` section in every Kubernetes object.

```yaml
...
metadata:
  annotations:
    example.com/icon-url: "https://example.com/icon.png"
...
```

## Service Discovery

Kubernetes is a dynamic system where Pods are placed on nodes, and the numbers of Pods running can vary based on load. This makes it easy to run a lot of things but you also need to be able to find those running things. This class of problem is referred to as **service discovery**. Service discovery tools help solve the problem of finding which processes are listening at which addresses for which services. 

### The Service object

In Kubernetes service discovery starts with a **Service object**. A Service object is a way to create a named label selector, but it has some other functionality.

To imperatively create a Service object use the command similar to `kubectl expose deployment alpaca-prod` (assuming you have the alpaca-prod) deployment already running. The `kubectl expose` command will pull both the label selector and the relevant ports from the deployment definition to set up the service. It will also assign a virtual IP (called **cluster IP**) to the service. This is a special IP address which Kubernetes will use to load balance across all the Pods that are identified by the selector.  This process of having Pods that match a selector get load balanced in the cluster IP is the service discovery mechanism (just my take away need to confirm)

You can get a list of services with the command `kubectl get services`

### Service DNS

Because the cluster IP is virtual, it is stable and it is appropriate to give it a DNS address. Kubernetes provides a DNS service exposed to Pods running in the cluster and provides DNS names to cluster IPs.

An example of the DNS name and how it breaks down is `alpaca-prod.default.svc.cluster.local` where...

* `alpaca-prod` is the name of the service in questions
* `default` is the namespace that the service is in
* `svc` is for recognizing that this is a service
* `cluster.local` is the base domain name for the cluster

When referring to a service in your won namespace you can just use the service name.  You can also refer to a service in another namespace by having the namespace in the specified name (`alpaca-prod.default` for example) or you can use the FQDN.

### Readiness Checks

One nice things the Service object does is track if a Pod is ready to accept requests or not. This is useful if your applications needs some time to initialize when it starts up. This is the readiness check functionality.  You can specify the readiness check in your manifest file with code similar to

```yaml
speck:
  ...
  template:
  ...
  spec:
    containers:
      ...
      name: alpaca-prod
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        periodSeconds: 2
        initialDelaySeconds: 0
        failureThreshold: 3
        successThreshold: 1
```

In the example above the readiness check will look for successful GET request to `/ready` endpoint on port 8080. It will check every 2 seconds starting as soon as the Pod comes up. If three successive checks fail then the Pod will be considered not ready. If one checks succeeds the Pod will again be considered ready.  Only ready pods are sent traffic.  This functionality is not just useful at startup its a also a good way for an overloaded or sick server to signal to the system that it doesn't want to receive traffic anymore, and is a good way to implement a graceful shutdown. Server can signal it no longer wants to receive traffic complete all jobs or connections and shut down.

### Exposing a service outside a cluster

You can use the `NodePort` functionality to have your service be accessible from any cluster node. Essentially if you specify `spec.type` as `NodePort` or use `--type=NodePort` when calling expose command Kubernetes will assign a port to your service that can be used from any cluster node to access the service.

The Service object operates at Layer 4 of the OSI model which means that it only forward TCP and UDP connections and doesn't look inside of those connections. 

#### Cloud Integration

If your cloud supports it, you can use the `LoadBalancer` type which builds on the `NodePort` concept. Essential the cloud provider will create a load balancer and direct it at nodes in your cluster. This functionality is cloud provider specific, but is a way to get your application exposed/usable to the world.

### Endpoints

Some applications (and the system itself) want to be able to use services without a cluster IP. This can be done with the **Endpoints** object. For every Service object Kubernetes creates a buddy Endpoints object that contains the IP address of that service.  You an view the Endpoints object with the command `kubectl describe endpoints alpaca-prod`. To use a service an application that is aware of Endpoints (likely an application written tow ork with Kubernetes) can talk to Kubernetes API directly to look up endpoints and call them. The Kubernetes API even has the capability to "watch" objects and be notified as soon as they change. This allows the client to react immediately as soon as the IPs associated with a service change. Most applications don't use this and just use stable IP addresses that don't change often, but the alternative is there.

### kube-proxy and cluster IPs

Cluster IPs are stable virtual IPs that load-balance traffic across all of the endpoints in a service. This is performed by a component running on every node in the cluster called `kube-proxy`. `kube-proxy` watches for new services in the cluster via the API server and then programs a set of `iptables` rules in the kernel of that host to rewrite the destinations of packets so they care directed at one of the endpoints for that service. If the set of endpoints for a service changes (due to Pods coming and going ro due to failed readiness checks) the set of `iptables` rules is rewritten.

The cluster IP itself is usually assigned by the API server as the service is created, however when creating the service the user can specify a specific cluster IP.  Once set the cluster IP cannot be modified without deleting and recreating the Service object.

### Connecting with other environments

 When you are connecting Kubernetes to legacy resources outside of the cluster, you can use selector-less services to declare a Kubernetes ser‐ vice with a manually assigned IP address that is outside of the cluster. That way, Kubernetes service discovery via DNS works as expected, but the network traffic itself flows to an external resource.

 Connecting external resources to Kubernetes services is somewhat trickier. If your cloud provider supports it, the easiest thing to do is to create an “internal” load balancer that lives in your virtual private network and can deliver traffic from a fixed IP address into the cluster. You can then use traditional DNS to make this IP address available to the external resource. Another option is to run the full kube-proxy on an external resource and program that machine to use the DNS server in the Kubernetes cluster. Such a setup is significantly more difficult to get right and should really only be used in on-premise environments. There are also a variety of open source projects (for example, Hashicorp’s Consul) that can be used to manage connectivity between in-cluster and out-of-cluster resources.

## HTTP Load Balancing with Ingress

There are some challenges with using `NodePort` based Service object to expose your service outside of the cluster. For one since `NodePort` Service objects operate at Layer 4, you will need to have clients connecting on to a unique port per service.  If you use a `LoadBalander` based Service object you will need to allocate an expensive cloud based load balancer outside your cluster for each service.

For Layer 7 (HTTP) applications we can do better. Outside of Kubernetes this problem is typically solved with a reverse proxy which decodes the incoming requests and sends them to the right upstream service. In Kubernetes this functionality is called **Ingress**.  Ingress is a Kubernetes native way to implement the virtual hosting pattern described above. This virtual hosting pattern requires the administrator to manage the load balancer configuration file. In a dynamic environment as the set of virtual hosts expands, this can be very complex. Ingress simplifies this by standardizing that configuration, moving it to a standard Kubernetes object, and merging multiple Ingress objects into a single config for the load balancer.

The ingress controller is a software system exposed outside the cluster using a service type: `LoadBalancer`. It then proxies requests to "upstream" servers.  The configuration of how it does this is the result of reading and monitoring Ingress objects.

### Ingress spec versus Ingress controllers

The implementation of Ingress is very different from other regular resource objects in Kubernetes. Ingress is split into a common resource specification and a controller implementation. There is not "standard" Ingress controller that is built into Kubernetes, and the user must install one of the many optional implementations. 

Users can create and modify Ingress objects just like every other object; but by default, there is not code running to actually oct on those objects. It is up to the users (or the distribution they are using) to install and manage an outside controller. In this way the, the controller is pluggable.

There are multiple reasons for this pluggable approach in Ingress. First there is no single HTTP load balancer that can universally be used. In addition to many software load balancers (both open source and proprietary), there are also load balancing capabilities provided by cloud providers such as ELB on AWS, and hardware based load balancers. The second reason is that Ingress was added to Kubernetes before the standard extensibility options. In the future Ingress may change to follow those new extensibility options.

### Configuring DNS

To make Ingress work well, you need to configure DNS entries to the external address of your load balancer. You can map multiple host names to a single external endpoint and the Ingress controller will direct incoming requests to the appropriate upstream service based on that hostname.

### Using hostname

The most common use of directing traffic based on properties of a request is using hostnames. The yaml file below defines an Ingress object that will route requests to `alpaca.example.com` to the alpaca service.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: host-ingress
spec:
  rules:
  - host: alpaca.example.com
  http: paths:
  - backend:
    serviceName: alpaca
    servicePort: 8080
```

#### Default http backends

Certain Ingress controllers use a concept of a `default-http-backend`. This is used if a request comes in that does not have any http backend defined.  The default backend is uses as the default.

### Paths

You can use Ingress to direct traffic based on not just he hostname, but also the path in the HTTP request.  We can do this by specifyng a path in the `paths` entry. In the example below, we are directing everything coming into `http://bandicoot.example.com` to the bandicoot service, but we send `http://bandicoot.example.com/a` to the alpaca service. This type of scenario can be used to host multiple services on different paths of a single domain.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: path-ingress
spec:
  rules:
  - host: bandicoot.example.com
  http:
    paths:
    - path: "/"
      backend:
        serviceName: bandicoot
        servicePort: 8080
    - path: "/a/"
      backend:
        serviceName: alpaca
        servicePort: 8080
```

When there are multiple paths on the same host listed in the Ingress system, the longest prefix matches. In the example above, traffic staring with /a/ is forwarded to the alpaca service, while all other traffic (starting with /) is directed to the bandicoot service.  As requests get proxied to the upstream service, the path remains unmodified. That means a request to `bandicoot.example.com/a/` shows up to the upstream server that is configured for that request hostname path.

### Advanced Ingress topics and gotchas

Many of Ingress' extended features are exposed via annotations on the Ingress object. Be careful, as these annotations can be hard to validate and are easy to get wrong. Many of these annotations apply to the entire Ingress object and so can be more general than you might like. To scope the annotations down you can always split a single Ingress object into multiple Ingress objects. The ingress controller should read them and merge them together.

#### Running multiple Ingress Controllers

If you are running multiple Ingress controllers on a single cluster you can control which Ingress object is meant for which ingress controller using the `kubernetes.io/ingress.class` annotation. The value should be a string that specifies which Ingress controller should look at this object. The ingress controllers themselves, then, should be configured wth that same string and should only respect those Ingress objects with the correct annotation. If `kubernetes.io/ingress.class` annotation is missing, behavior is undefined. Its likely multiple controllers will fight to satisfy the Ingress and write the status field of the Ingress objects.

#### Multiple Ingress Objects

If you specify multiple Ingress objects, the Ingress controllers should read them all and try to merge them into a coherent configuration. If duplicate or conflicting configurations are specified, the behavior is undefined.

#### Ingress and namespaces

For security an Ingress object can only refer to an upstream service in the same namespace. This means that you can't use an Ingress object to point a subpath to a service in another namespace. On the other hand multiple multiple Ingress objects in the different namespaces can specify sub-paths for the same host. These Ingress objects are then merged together to come up with the final config for the Ingress controller.  This cross-namespace behavior means that it is necessary that Ingress be coordinated globally across the cluster; otherwise an Ingress object in one namespace can cause issues for another namespace. Advanced users may try to force controls on this using admission controller, but there are no restrictions out of the box.

#### Path rewriting

Some ingress controller implementations support, optionally, doing path rewriting. This can modify the path in the HTTP request as it gets proxied. This is typically specified with an annotation ont he Ingress controller (check the documentation for controller you are using). Be careful with path rewriting as if your web applications tries to link within itself using absolute paths the rewrite may cause issues.

#### Serving TLS

To serve TLS via Ingress you will need to specify the certificate and key.  You can create the secret using `kubectl create secret tls <secret-name> --cert <certificate-pem-file> --key <private-key-pem-file>`. Once you have the certificate uploaded, you can reference it in an Ingress object. If multiple Ingress objects specify certificates for the same hostname, the behavior is undefined.

Check out [cert-mager](https://github.com/cert-manager/cert-manager) for setting up an API driven local certificate authority.

```yaml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
    - hosts:
      - alpaca.example.com
      secretName: tls-secret-name
rules:
- host: alpaca.example.com
  http:
    paths:
    - backend:
      serviceName: alpaca
      servicePort: 8080
```

#### Alternate Ingress Implementations

Cloud providers have their own L7 based Ingress implementation that exposes the specific cloud based load balancer.  Instead of configuring the lod balancer to run in a Pod, these controllers take Ingress objects and use them to configure the cloud based load balancer via an API. This reduces load on the cluster, but you need to pay for the load balancer resources used.

The most popular generic Ingress controller is probably the open source [NGINX Ingress controller](https://github.com/kubernetes/ingress-nginx/). There is also a commercial version based on this project.   [Embassador](https://github.com/emissary-ingress/emissary) and [Gloo](https://github.com/solo-io/gloo) are options you should look at if you are looking to build an API gateway.  [Traefik](https://traefik.io) is a reverse proxy that can also function as an Ingress controller. It has a set of features and dashboards that are very developer-friendly.

## ReplicaSets

In real world use cases you rarely run a single Pod. Typically multiple pods are run for redundancy, scale, sharding or other reasons. A replicated set of Pods should be a single entity (to prevent from repetitive work). The **ReplicaSet** object in Kubernetes acts as a cluster wide Pod manager, ensuring that the right types and numbers of Pods are running at all times. ReplicaSets not Pods tend to be the building blocks of applications in Kubernetes. ReplicaSets provide self-healing for our applications at the infrastructure level. Pods managed by ReplicaSets are automatically rescheduled under certain failure conditions such as node failures and network partitions.

### Adopting exiting containers

Though ReplicaSets create and manage Pods, they do not own the Pods they create. ReplicaSets use label queries to identify the set of Pods they should be managing. They then use the general Kubernetes APIs to create the Pods they are managing. This decoupling of Pods and ReplicaSets supports several important behaviors.  Because ReplicaSets are decoupled from the Pods they manage, you can create a ReplicaSet that will "adopt" an existing Pod and scale out additional copies of those containers. This is a path to go from a single imperatively declared Pod to a replicated set of Pods managed by ReplicaSet.

### Quarantining Containers

When a server misbehaves, Pod-level health checks will automatically restart the Pod. If your health checks are incomplete, a Pod can be misbehaving, but still be a part of the replicated set. In this scenario you can kill the Pod, but then developers would only have logs to troubleshoot with.  A better option is to leave the Pod running but quarantine it so that it is not sent traffic. You can do this modifying the set of labels on the Pod. This will disassociate it from the ReplicaSet, but leave it running for developers to troubleshoot.

### Designing with ReplicaSets

ReplicaSets are designed to represent a single, scalable microservice inside your architecture. The key characteristic of ReplicaSets is that every Pod that is created by the ReplicaSet controller is entirely homogenous. Typically, these Pods are then fronted by a Kubernetes service load balancer, which spreads traffic across the Pods that make up the service. Generally speaking ReplicaSets are designed for stateless (or nearly stateless) services. When it is scaled down an arbitrary Pod wil be selected for deletions and this should not impact your application.

### ReplicaSet spec

All ReplicaSets must have a unique name (defined using the `metadata.name` field), a `spec` section that describe the number of Pods (replicas) that should be running in the cluster, and a Pod template that describes the Pod to be created.

Below is an example of a minimal ReplicaSet

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

### Inspecting a ReplicaSet

You can use the command `kubectl describe rs <ReplicaSet name>` to get details about a running ReplicaSet

### Finding a ReplicaSet from a Pod

The ReplicaSet controller adds an annotation to every Pod that it creates. The key for the annotation is `kubernetes.io/created-by` If you run the command `kubectl get pods <pod-name> -o yaml` it will provide the ReplicaSet that is managing the Pod. Note that these annotations are best effort and while added when the Pod is created could be removed by a Kubernetes user at any time.

### Finding a set of Pods for a ReplicaSet

You can determine the Pods managed by a ReplicaSet by using a label query. You can use `kubectl describe` to find the set of labels from the ReplicaSet. Then do a label query such as `kubectl get pods -l app=kuard,version=2` to find the Pods. This is the same query that ReplicaSet executes to determine the current numbers of Pods.

### Scaling ReplicaSets

Replica sets are scaled up or down by updating the `spec.replicas` key on the ReplicaSet object stored in Kubernetes. When a ReplicaSet is scaled up, new Pods are submitted to the Kubernetes API using the Pod template defined on the ReplicaSet.

#### Imperative scaling ReplicaSets with kubectl

`kubectl scale replicasets <ReplicaSet name> --replicas<numer>` for example `kubectl scale replicasets kuard --replicas=4`

This works, and can be useful for demos or troubleshooting, but in typical operations declarative approach should be used.

#### Declaratively scaling ReplicaSets with kubectl apply

The declarative approach is to make changes in a version controlled file and then applying those changes to our cluster with the command `kubectl apply -f <manifest_file.yaml>` for example `kubectl apply -f kuard-rs.yaml`

#### Autoscaling a ReplicaSet

Kubernetes supports scaling ReplicaSets using **HPA (Horizontal Pod Autoscaling)**. HPA requires the presence of a `heapster` Pod on your cluster which keeps track of metrics and provides an API for consuming those metrics that HPA uses to make decisions. Most Kubernetes installations include `heapster` by default.

##### Side not on scaling

Kubernetes does not currently support vertical scaling (adding CPU for example) but this is planned. Many solution offer cluster auto scaling where extra nodes are added as needed.

#### Autoscaling based on CPU

Scaling based on CPU is the most common  use case for Pod autoscaling. Its most useful for request-based systems that consume CPU proportionally to the number of requests they are receiving while using a relatively static amount of memory.

Use the command `kubectl autoscale rs kuard --min=2 --max=5 --cpu-percent=80`

To view, modify or delete this resource you can use the standard kubectl commands and the `horizontalpodautoscalers` resource (you can shorten that to `hpa`) for example `kubectl get hpa`.

Because of the decoupled nature of Kubernetes, there is no direct link between HPA and the ReplicaSet. This leads to some anti patterns. Its a bad idea to combine both autoscaling and imperative or declarative management of the number of replicas. If both you and an autoscaler are attempting to modify the number of replicas, it's highly likely that you will clash, resulting in unexpected behavior.

#### Deleting ReplicaSets

When a ReplicaSet is no longer required, it can be deleted using the `kubectl delete` command. For examples `kubectl delete rs kuard`. By default this also deletes the Pods that are managed by the ReplicaSet.  If you don't want to delete the Pods use the command similar to `kubectl delete rs kuard --cascade=false`

## Deployments

The Deployment object exists to manage the release of new versions. Deployments enable you to easily move from one version of your code to the next.

Like with all Kubernetes objects, a deployment can be represented as a declarative YAML object that provides the details about what you want to run. Below is a manifest for a single instance of the `kuard` application.

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: kuard
spec:
  selector:
    matchLabels:
      run: kuard
    replicas: 1
    template:
      metadata:
        labels:
          run: kuard
      spec:
        containers:
        - name: kuard
          image: gcr.io/kuar-demo/kuard-amd64:blue
```

To run the deployment specified by the above script you would run the command `kubectl create -f kuard-deployment.yaml`

### Deployment internals

Similar to how ReplicaSets manage Pods; deployments manage ReplicaSets. As with all relationships in Kubernetes the deployment to ReplicaSets relationship is defined by labels and a label selector. You can see the label selector by looking at the Deployment object with the below command.

`kubectl get deployments kuard -o jsonpath --template {.spec.selector.matchLabels}`

You can imperatively scale a deployment using the command `kubectl scale deployments kuard --replicas=2`.  ***Note:*** If you want to take any actions on the ReplicaSets managed by a deployment you need to delete the deployment. Otherwise the self healing nature of Kubernetes will keep taking actions to align the ReplicaSets with the stored specification for the deployment.

### Creating deployments

As a starting point you can generate the YAML manifest for an imperatively declared deployment to create a declarative onw.

To get the deployment as a YAML file use the following command...

`kubectl get deployments kuard --export -o yaml > kuard-deployment.yaml` followed by `kubectl replace -f kuard-deployment.yaml --save-config`

The reason you need to run the second command (with --save-config) is that it adds an annotation so that when applying changes in the future, kubectl will know what the last applied configuration was for smarter merging of conflicts.  If you always use `kubectl apply`, this step is only required after the fist time you create a deployment using `kubectl create -f`.

The deployment spec has a very similar structure to the ReplicaSet sepc. There is a Pod template which contains a number of containers that are created for each replica managed by the deployment. In addition to the Pod specification, there is also a `strategy` object.

```yaml
...
strategy:
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
  type: RollingUpdate
...
```

The `strategy` object dictates the different ways in which a rollout of the new software can proceed. There are two strategies supported: `Recreate` and `RollingUpdate` (discussed later in book)

### Managing deployments

To get information about a deployment use the command similar to `kubectl describe deployments kuard`

From the output of the above command, two of the most important pieces of information are `OldReplicaSets` and `NewReplicaSets`. These fields point to the ReplicaSet object this deployment is currently managing. if a deployment is in the middle of a rollout, both fields will be set to a value. If a rollout is complete, `OldReplicaSets` will be set to `<none>`.

You can also use the command `kubectl rollout history` to obtain the history of rollouts associated with a particular deployment. If you have a current deployment in progress you can use `kubectl rollout status` to obtain the current status of a rollout.

### Updating deployments

#### Scaling a deployment

To scale up a deployment, you would edit your YAML file to increase the number of replicas. Once you have saved and committed the change, you can use the `kubectl apply` command to put it into effect.

#### Updating a container image

Updating a container is also an update to a YAML file and a `kubectl apply` command call. It is a good idea to add an annotation to the deployment to record some information about the update.  Make sure to add this annotation to the template and not the deployment in the YAML file as `kubectl apply` command uses the field in the Deployment object. Also do not update the `change-cause` annotation when scaling as a change to that annotation will trigger a new rollout.

Once you make the change and call `kubectl apply` you can monitor the rollout via the `kubectl rollout` command.

If you are in the middle of a rollout and you want to temporarily pause the rollout (if you are seeing issues and want to investigate for example) you can use the command similar to `kubectl rollout pause deployments kuard`. To resume the rollout use `kubectl rollout resume deployments kuard`.

#### Rollout history

You can see the deployment history by running the command `kubectl rollout histoyr deployment kuard` If you are interested in more details about a particular revision, you can add the `--revision` flag as in `kubectl rollout history deployment kuard --revision=2`

If you need to undo a rollout to the prior version you can use the command `kubectl rollout undo deployment kuard`.  The undo command works regardless of the state of the rollout. You can undo both partially completed and fully completed rollouts. ***Note***: This is the imperative approach. The preferred declarative way is to update the YAML file to the version you are rolling back to and run that deployment.

By default the revision history of a deployment is kept attached to the Deployment object. If you have deployment that you plan to keep for a long time and you make frequent updates you should limit the revision history. You can do this by setting the `revisionHistoryLimit` property in the deployment specification.

### Deployment strategies

As mentioned prior, Kubernetes supports two different rollout strategies, `Recreate` and `RollingUpdate`

#### Recreate Strategy

The recreate strategy simply updates the ReplicaSet it manages to us the new image and terminates all of the Pods associated with the deployment. The ReplicaSet notices that it no longer has any replicas, and re-creates all Pods using the new image. Once the pods are re-created, they are running the new version.  This strategy will almost certainly result in some downtime. Its typically uses for test deployments.

#### RollingUpdate strategy

RollingUpdate can be used to roll out a new version of your service while it is still receiving user traffic, without any downtime. It works by updating a few Pods at a time until all of the Pods are running the new version of your software.

This means that for a period of time, both the new and the old version of your service will be receiving and serving traffic. The software you are deploying needs to support this state.

##### Configuring a rolling update

There are two parameters you can use to tune the rolling update behavior; `maxUnavailable` and `maxSurge`.

The `maxUnavailable` parameter sets the maximum number of Pods that can be unavailable during a rolling update. It can either be set to an absolute number (3 for example meaning maximum of 3 Pods can be unavailable) or as a percentage (20% meaning a maximum of 20% of the desired number of replicas can be unavailable).  Generally you would use percentage, but setting hard value of 1 lets you deploy one Pod at a time.

Using reduced capacity is one options of doing a deployment, but it may not always be a valid option (if you do not have dips in traffic for example).  The other option is to create extra instances and take out the old ones which is done with the `maxSurge` parameter. The `maxSurge` parameter controls how many extra resources can be created to achieve a rollout.  For example you can set `maxUnavailable` to 0 and `maxSurge` to 20%. The rolling update will scale up by 20% first then start removing the old instances.

Setting `maxSurge` to 100% is equivalent to a blue/green deployment. The deployment controller first scales the new version to 100% of the old version. Once the new version is healthy, it immediately scales the old version down to 0%

##### Slowing rollouts to ensure service health

The deployment controller examines the Pod's status with readiness checks. If you want to use deployments to reliably roll out your service you must implement health checks for the containers in your Pod. Without these checks deployment controller is running blind.

Just because a pod comes up healthy does not mean it does not have any issues. Memory leaks or low frequency bugs can take some time to be noticed. In most real world scenarios you want to wait some time before moving on to the next Pod. For deployments this time to wait can be defined by the `minReadySeconds` parameter.  Setting `minReadySeconds` to 60 means a deployment has to wait for 60 seconds after a Pod comes up healthy until it moves on to the next one.

You also want to set a timeout for the rollout should you have a scenario where Pods are just not coming up healthy. If you are using automated system to deploy a timout is a good place to fire off a ticket and roll back the deployment. To set a timeout use the `progressDeadlineSeconds` parameter. Its important to note that his setting is for deployment progress not the overall length of a deployment. Every time the deployment creates or deletes a Pod the clock is reset for the timeout.

#### Deleting a Deployment

To imperatively delete a deployment ue the command similar to `kubectl delete deployment kuard`.  To declaratively delete a deployment `kubectl delete -f kuard-deployment.yaml`.

Deleting a deployment deletes the entire service including the ReplicaSets and Pods. If you want to leave those intact you can use the `--cascade=false` option to only delete the Deployment object.

#### Monitoring a deployment

The status of a deployment can be obtained from the `status.conditions.array` where there will be a `Condition` whose `Type` is `Progressing` and whose `Status` is `False`. A deployment in such a state has failed and will not progress further.  How long you should wait for this state is controlled by the `spec.progressDeadlineSeconds` property.

## DaemonSets

DaemonSets are a construct that is used to make sure that a Pod is running across a set of nodes in a Kubernetes cluster. DeamonSets are used to deploy daemons such as log collectors and monitoring agents, which typically need to run on every node. You can use labels to run DaemonSet Pods on specific nodes; for example you may want to run special intrusion detection software on nodes that are exposed tot he edge network. You can also use DaemonSets to install software on nodes in a cloud-based cluster. If you haver a need to have specific software on every node of your cluster DaemonSets is the way to accomplish this.  You can even mount the host filesystem and run scripts that install RPM/DEB packages onto the host operating system.  DaemonSets are particularly useful on auto scaled Kubernetes clusters.

### DaemonSet Scheduler

By default, a DaemonSet will create a copy of a Pod on every node unless a node selector is used, which will limit eligible nodes to those with a matching set of labels. DaemonSets determine which node a Pod will run on at Pod creating time by specifying the `nodeName` field in the Pod spec. As a result, Pods created by daemonSets are ignored by the Kubernetes scheduler and are driven by the DaemonSet controller. If a new node is added tot he cluster, the DaemonSet controller will notice that it is missing a Pod and will add the Pod to the new node.

### Creating DaemonSets

Below is an example of creating a `fluentd` logging agent on every node in the cluster.

DaemonSets require a unique name across all DaemonSets in a given Kubernetes namespace. Each DaemonSet must include a Pod template spec, which will be used to create Pods as needed.

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: fluentd
  labels:
    app: fluentd
spec:
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v0.14.10
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

### Limiting DaemonSets to specific nodes

The first step in limiting DaemonSets to specific nodes is to add the desired set of labels to a subset of nodes. This can be achieved using the `kubectl label` command. Fore example the command below adds the `ssd=true` label to a single node:

`kubectl label nodes k0-default-pool-35609c18-z7tb ssd=true`

Using a label selector, we can filter nodes based on labels. The command below lists nodes which have the `ssd` label set to `true`.

`kubectl get nodes --selector ssd=true`

### Node selectors

Node selectors can be used to limit what nodes a Pod can run on in a given Kubernetes cluster. Node selectors are defined as part of the Pod spec when creating a DaemonSet. The DaemonSet configuration in the example below limits NGINX to running only on nodes with the `ssd=true` label set.

```yaml
apiVersion: extensions/v1beta1
kind: "DaemonSet"
metadata:
  labels:
    app: nginx
    ssd: "true"
  name: nginx-fast-storage
spec:
  template:
    metadata:
      labels:
        app: nginx
        ssd: "true"
    spec:
      nodeSelector:
        ssd: "true"
      containers:
        - name: nginx
          image: nginx:1.10.0
```

***Note:*** Removing labels from a node that are required by a DaemonSet's node selector will cause the Pod being managed by that DaemonSet to be removed from the node.

### Updating a DaemonSet

Prior to Kubernetes 1.6 the only way to update Pods managed by a DaemonSet was to update the DaemonSet and then manually delete each Pod that was managed by the DaemonSet so that it would be recreated with the new configuration. With version 1.6, DaemonSets gained an equivalent to the Deployment object that manages a DaemonSet rollout.

#### Rolling update of a DamoneSet

Daemon sets can be rolled out using the same `RollingUpdate` strategy that deployments use. You can configure the update strategy using the `spec.updateStategy.type` field, which should have the value `RollingUpdate`. When a DaemonSet has an update strategy of `RollingUpdate`, any change to the `spec.template` field (or subfields) in the DaemonSet will initiate a rolling update.

There are two parameters that control the rolling update of a DaemonSet.

* `spec.minReadySeconds` - determines how long a Pod must be "ready" before the rolling update proceeds to upgrade subsequent Pods. Its good practice to set this to a reasonable long value of 30 to 60 seconds to make sure the Pod comes up healthy.
* `spec.updateStrategy.rollingUpdate.maxUnavailable` - indicates how many Pods may be simultaneously updated by the rolling update. Setting this to 1 is a safe approach as only one node of the cluster at a time will be impacted.

Once a rolling update has started you can use the `kubectl rollout` command to see the current status of a DaemonSet rollout. For example `kubectl rollout status daemonSets my-daemon-set`.

### Deleting a Daemon Set 

`kubectl delete -f fluentd.yaml`

***Note:*** Deleting a DaemonSet will also delete all the Pods being managed by that DaemonSet. You can use the `--cascade=false` option to prevent that from happening.

## Jobs

The Job construct in kubernetes lets you run one time short-lives task. A job creates Pods that run until successful termination (i.e. exit with 0). 

### The Job object

The Job object is responsible for creating and managing Pods defined in a template in the job specification. These Pods generally run until successful completion. The Job object coordinates running a number of Pods in parallel. If the Pod fails before a successful termination, the job controller will create a new Pod based on the pod template in the job specification.

Given that Pods have to be scheduled, theres is a chance that your job wil not execute if the required resources are not found by the scheduler. Also, due to the nature of distributed systems there is a small chance, during certain failure scenarios, that duplicate Pods will be created for a specific task.

### Job patterns

Jobs are designed to manage batch-like workloads where work items are processed by one or more Pods. By default, each job runs a single Pod once until successful termination. This job pattern is defined by two primary attributes of a job, namely the number of job completions and the number of Pods to run in parallel. In the case of run once until completion for example, the `completions` and `parallelism` parameters are set to 1.

| Job pattern type | Use case | Behavior | Completions | Parallelism |
| ---------------- | -------- | -------- | ----------- | ----------- |
| One shot | Database migrations | A single Pod running once until successful completion | 1 | 1 |
| Parallel fixed comletions | Multiple pods processing a set of work in parallel | Once or more Pods running one or more times until reaching a fixed completion count | 1+ | 1+ |
| Work queue | Multiple Pods processing from a centralized work queue | One or more Pods running once until successful termination | 1 | 2+ |

#### One shot

One shot jobs provide a way to run a single Pod once until successful termination. A Pod must be created and submitted to the Kubernetes API. This is done using a Pod template defined in the job configuration. Once a job is up and running, the Pod backing the job must be monitored for successful termination. A job can fail for any number of reasons, including an application error, an uncaught exception during runtime, or a node failure before the jbo has a chance to complete. In all cases, the job controller is responsible for recreating the Pod until a successful termination occurs. 

There are multiple ways to create a one-shot job in Kubernetes. The easiest is to use the `kubectl` command line tools for example...

`kubectl run -i oneshot --image=gcr.io/kuar-demo/kuard-amd64:blue --restart=OnFailure -- --keygen-enable --keygen-exit-on-complete --keygen-num-to-gen 10`

Some things to note about the above example

* The `-i` option indicates that this is an interactive command. `kubectl` will wait until the job is running and then show the log output from the first (and in this example only) Pod in the job.
* `--restart=OnFailure` is the option that tells `kubectl` to create a Job object.  Restart policy can be set to Never if you do not wish for kubernetes to keep trying to restart the Pod if it fails. The behavior with Never will be to keep crating new Pods which can create a lot of Junk in your cluster so the `OnFailre` restart option is preferred. Kubelet has mechanisms to do a crash loop backoff so as to not keep eating resources on the cluster.
* All of the options after `--` are command line arguments to the container image. These instruct our test server (kuard) to genearte 10 4096 bit SSH keys and then exit.
* `kubectl` often misses the first couple of lines of the output with the `-i` option.

After the job has completed, the Job object and related Pod are still around. This is so that you can inspect the log output. Note that this job won't show up in `kubectl get jobs` unless you pass the `-a` flag. Without this flag `kubectl` hides completed jobs. To delete the job use the command `kubectl delete jobs oneshot`.

You can also use a manifest file to create a job.  Below is an example.  You would then submit the job with the command `kubectl apply -f job-oneshot.yaml`.  You can view the results of the job by looking at the logs of the Pod that was created using `kubectl describe jobs oneshot` to find the Pod and `kubectl logs oneshot-4kfdt` to see the logs from Pod.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: oneshot
spec:
  template:
    spec:
      containers:
      - name: kuard
        image: gcr.io/kuar-demo/kuard-amd64:blue
        imagePullPolicy: Always
    args:
    - "--keygen-enable"
    - "--keygen-exit-on-complete"
    - "--keygen-num-to-gen=10"
    restartPolicy: OnFailure
```

One thing to consider is a job may get stuck and not make progress. To handle this you want to use a liveness probe to determine if the job should be restarted.

### Parallelism

If for example you wnt to generate 100 keys by having 10 runs of kuard with each generating 10 keys while limiting to just running 5 pods at a time this will translate to `completions` of 10 and `parallelism` of 5.  Below is an example of such a job manifest.  You can start this job with the command `kubectl apply -f job-parallel.yaml`

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel
  labels:
    chapter: jobs
spec:
  parallelism: 5
  completions: 10
  template:
    metadata:
      labels:
        chapter: jobs
    spec:
      containers:
      - name: kuard
        image: gcr.io/kuar-demo/kuard-amd64:blue
        imagePullPolicy: Always
        args:
        - "--keygen-enable"
        - "--keygen-exit-on-complete"
        - "--keygen-num-to-gen=10"
      restartPolicy: OnFailure
```

### Work Queues

The manifest example below is telling the job to start up fie Pods in parallel. As the `completions` parameter is unset, we put the job into a worker pool mode. Once the first Pod exits with a zero exit code, the job will start winding down and will not start any new Pods. This means that none of the workers should exit until the work is done and they are all in the process of finishing up.  Note that in the book a work to be done queue was set up for the example workers.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  labels:
    app: message-queue
    component: consumer
    chapter: jobs
  name: consumers
spec:
  parallelism: 5
  template:
    metadata:
      labels:
        app: message-queue
        component: consumer
        chapter: jobs
    spec:
      containers:
      - name: worker
        image: "gcr.io/kuar-demo/kuard-amd64:blue"
        imagePullPolicy: Always
        args:
        - "--keygen-enable"
        - "--keygen-exit-on-complete"
        - "--keygen-memq-server=http://queue:8080/memq/server"
        - "--keygen-memq-queue=keygen"
      restartPolicy: OnFailure
```

For the example manifest above we can clean up using labels with the command `kubectl delete rs,svc,job -l chapter=jobs`

### CronJobs

Below is an example of a CronJob definition in Kubernetes. Note the `sepc.schedule field`, which contains the interval for the CronJob in standard cron format. As with other definitions you can schedule it with the command `kubectl create -f cron-job.yaml` and use `kubectl describe <cron-job>` to get the details.

```yaml
apiVersion: batch/v1beta1
    kind: CronJob
    metadata:
      name: example-cron
    spec:
      # Run every fifth hour
      schedule: "0 */5 * * *"
      jobTemplate:
        spec:
          template:
            spec:
              containers:
              - name: batch-job
                image: my-batch-image
              restartPolicy: OnFailure
```

## ConfigMaps and Secrets

### ConfigMaps

ConfigMaps are used to provide configuration information for workloads. This information can be a short string or a file. One way to think of ConfigMaps is a Kubernetes object that defines a set ov variables that can be used when defining the environment or command line for your containers.

### Creating ConfigMaps

To imperatively create a ConfigMap which makes a `my-config.txt` file available to a Pod you would use a command similar to ...

`kubectl create configmap my-config --from-file=my-config.txt --from-literal=extra-param=extra-value --from-literal=another-param=another-value`

In this example the context of `my-config.txt` are below...

```text
# This is a sample config file that I might use to configure an application
parameter1 = value1
parameter2 = value2
```

The YAML for the above command would ...

```yaml
apiVersion: v1
    data:
      another-param: another-value
      extra-param: extra-value
      my-config.txt: |
        # This is a sample config file that I might use to configure an application
        parameter1 = value1
        parameter2 = value2
    kind: ConfigMap
    metadata:
      creationTimestamp: ...
      name: my-config
      namespace: default
      resourceVersion: "13556"
      selfLink: /api/v1/namespaces/default/configmaps/my-config
      uid: 3641c553-f7de-11e6-98c9-06135271a273
```

You can get the YAML for the ConfigMap imperatively created with the command `kubectl get configmaps my-config -o yaml`

### Using a ConfigMap

There are three main ways to use a ConfigMap:

* Filesystem - You can mount a ConfigMap into a Pod. A file is created for each entry based on the key name. The contents of that file are set to the value.
* Environment variable - A ConfigMap can be used to dynamically set the value of an environment variable
* Command line arguments - Kubernetes supports dynamically creating the command line for a container based on ConfigMap values.

The manifest below shows examples of these three use cases.

For the filesystem method, we create a new volume inside the Pod and give it the name `config-volume`. We then define this volume to be a ConfigMap volume and point at the ConfigMap to mount. We have to specify where this gets mounted into the `kuard` containers with a `volumeMount`. In this case we are mounting it a `/config`. Environments variables are specified with a special `valueFrom` member. This references the ConfigMap and the data key to use within that ConfigMap. Command line arguments build on environment variables. Kubernetes will perform the correct substitution with a special `$(<env-var-name>)` syntax.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard-config
spec:
  containers:
    - name: test-container
      image: gcr.io/kuar-demo/kuard-amd64:blue
      imagePullPolicy: Always
      command:
        - "/kuard"
        - "$(EXTRA_PARAM)"
      env:
        - name: ANOTHER_PARAM
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: another-param
        - name: EXTRA_PARAM
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: extra-param
      volumeMounts:
        - name: config-volume
          mountPath: /config
  volumes:
    - name: config-volume
      configMap:
        name: my-config
  restartPolicy: Never
```

## Secrets

Secrets are similar to ConfigMaps but focus on making sensitive available to the workload. They can be used for things like credentials or TLS certificates. Secrets are exposed to Pds via explicit declaration in Pod manifests and the Kubernetes API. In this way, the Kubernetes secrets API provides an application centric mechanism for exposing sensitive configuration information to applications in a way that's easy to audit and leverages native OS isolation primitives.

***Note:*** By default, Kubernetes secrets are stored in plain text in `etcd` storage for the cluster. This may not be sufficient security depending on your requirements. Anyone who has cluster admin rights will be able to read all of the secrets in the cluster. Recent versions of Kubernetes add support for encrypting the secrets with a user supplied key which is generally integrated into a cloud key store. Most cloud key stores have integrations with Kubernetes flexible volumes, enabling you to skip Kubernetes secrets entirely and rely on the cloud providers key store.

### Creating Secrets

Secretes hold one or more data elements as a collection of key/value paris and are creted either with Kubernetes API or the kubectl command-line tool. 

Below is an example of using the `kubectl` command to created a secret which stored a TLS certificate and key from files local on the system where the command is being run. The command will create a secret with two elements `kuard.crt` and `kuard.key`

`kubectl create secret generic kuard-tls --from-file=kuard.crt --from-file=kuard.key`

To view the secret use the command below

`kubectl describe secrets kuard-tls`

### Consuming Secrets

Secrets can be consumed using the Kubernetes REST API by applications that know how to call that API directly. This approach may not be desirable as this makes your application not portable (will only work in Kubernetes). Another option is to use a **secrets volume**.

#### Secrets volumes

Secret data can be exposed to Pods using the secrets volume type. Secret volumes are managed by the `kubelet` and are created at Pod creation time. Secrets are stored on tmpfs volumes (aka RAM disks), and as such are not written to disk on nodes. Each element of a secret is stored in a separate file under the target mount point specified in the volume mount. 

The below Pod manifest demonstrates how to declare a secrets volume which exposes the `kuard-tls` secret volume to `/tls` mount point.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard-tls
spec:
  containers:
    - name: kuard-tls
      image: gcr.io/kuar-demo/kuard-amd64:blue
      imagePullPolicy: Always
      volumeMounts:
      - name: tls-certs
        mountPath: "/tls"
        readOnly: true
  volumes:
    - name: tls-certs
      secret:
        secretName: kuard-tls
```

### Private Docker registries

A special use case for secretes is to store access credentials for private Docker registries. Private Docker images can be stored across one or more private registries. This presents a challenge for managing credentials for each private registry on every possible node in the cluster. **Image pull secrets** leverage the secrets API to automate the distribution of private registry credentials. Image pull secrets are stored just like normal secrets but are consumed through the `spec.imagePullSecrets` Pod specification field.

Use the `create secret docker-registry to create this special kind of secret. For example `kubectl create secret docker-registry my-image-pull-secret --docker-username=<username> --docker-password=<password> --docker-email=<email-address>`

You then enable access to the private repository by referencing the image pull secret in the Pod manifest file as in the example below.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard-tls
spec:
  containers:
    - name: kuard-tls
      image: gcr.io/kuar-demo/kuard-amd64:blue
      imagePullPolicy: Always
      volumeMounts:
      - name: tls-certs
        mountPath: "/tls"
        readOnly: true
  imagePullSecrets:
  - name:  my-image-pull-secret
  volumes:
    - name: tls-certs
      secret:
        secretName: kuard-tls
```

If you are repeatedly pulling from the same registry, you can add the secrets to the default service account associated with each Pod to avoid having to specify the secrets in every Pod you create.

### Naming Constraints

The key names for data items inside of a secret or ConfigMap are defined to map to valid environment variable names. They may begin with a dot followed by a letter or number.  Following characters include dots, dashes, and underscores. Dots cannot be repreated and dots and underscores or dashes cannot be adjacent to each other. The below regular expression checks for valid names.

`^[.[?[a-zAZ0-9[([.[?[a-zA-Z0-9[+[-_a-zA-Z0-9[?)*$`

Below are some examples of valid and invalid names.

| Valid key name | Invalid key name |
| -------------- | ---------------- |
| `.auth_token` | `Token..properties`|
| `Key.pem` | `auth file.json` |
| `config_file` | `_password.txt` |

When selecting a key name, consider that these keys can be exposed to Pods via a volume mount. Pick a name that is going to make sense when specified on a command line or in a config file. Storing a TLS key as key.pem is more clear than `tls-key` when configuring applications to access secrets.

As of Kubernetes 1.6 ConfigMaps are unable to store binary data and can only be UTF-8 text.  You can in theory encode binary data into a base64 string and put it into the YAML file, but that makes YAML files hard to manage.

Maximum size for a ConfigMap or secret is 1MB.

### Managing ConfigMaps and Secrets

Secrets and ConfigMaps are managed through the Kubernetes API. The usual `create`, `delete`, `get` and `describe` commands work for manipulating these objects.

#### Listing

`kubectl get secrets`

`kubectl get configmaps`

`kubectl describe configmap my-config`

#### Creating

The easiest way to create a secret or ConfigMap is via `kubectl create secret generic` or `kubectl create configmap`. Ther are a variety of ways to specify the data items that go into the secret or ConfigMap. These can be combined in a single command.

* `--from-file=<filename>` - Load from the file with the secret data key the same as the filename

* `--from-file=<key>=<filename>` - Load from the file with the secret data key explicitly specified

* `--from-file=<directory>` - Load all files in the specified directory where the filename is an acceptable key name

* `--from-literal=<key>=<value>` - Use the specified key/value pair directly

#### Updating

You can update a ConfigMap or secret and have it reflected in running programs. There is no need to restart if the application is configured to reread configuration values. This is a rare feature but might be something you can add to your applications.

Below are three ways to update ConfigMaps or secrets.

##### Update from file

If you have a manifest for your ConfigMap or secret, you can just edit it directly and push a new version with `kubectl replace -f <filename>` You can also use `kubectl apply -f <filename>` if you previously created the resource with kubectl apply. Due to the way that datafiles are encoded into these objects, updating a configuration can be a bit cumbersome as there is no provision in `kubectl` to load data from an external file. The data must be stored directly in the YAML manifest. The most common use case is when the ConfigMap is defined as part of a directory or list of resources and everything is created and updated together. Oftentimes these manifests will be checked into source control. Be careful not put push secrets into a public location.

##### Recreate and update

If you store the inputs into your ConfigMaps or secrets as separate files on disk (as opposed to embedded into YAML directly), you can use `kubectl` to recreate the manifest and then use it to update the object. This will look something like...

`kubectl create secret generic kuard-tls --from-file=kuard.crt --from-file=kuard.key --dry-run -o yaml | kubectl replace -f -`

This command line first creates a new secret with the same name as our existing secret. If we just stopped there, the Kubernetes API server would return an error complaining that we are tying to create a secret that already exists. Instead, we tell `kubectl` not to actually send the data to the server but instead to dump the YAML that it *would have* sent to the API server to `stdout`. We then pipe that to `kubectl` replace and use `-f -` to tell it to read from `stdin`. In this way we can update a secret from files on disk without having to manually base64 encode the data.

##### Edit current version

You can use `kubectl edit` to bring up a version of the ConfigMap in your editor so you can modify it (you can also do this with a secret but you will have to update the base64 encoding value manually)

`kubectl edit configmap my-config`

One you make the desired changes in your editor and save, the new version of the object will be pushed to the Kubernetes API server.

##### Live updates

Once a ConfigMap or secret is updated using the API, it will automatically be pushed to all volumes that use that ConfigMap or secret. This update may take a few seconds. Currently there is not built in way to signal an application when a new version of ConfigMap is deployed. It is up to the application or some helper script to look for the config files to change and reload them.

## Role based access control for Kubernetes

Role Based Access Control (RBAC) has been generally available since Kubernetes 1.8. It provides a mechanism for restricting access and actions on Kubernetes APIs to ensure that only appropriate users have access to APIs in the cluster. RBAC is only part of a good security solution. Anyone who can run arbitrary code inside the Kubernetes cluster can effectively obtain root privileges on the entire cluster. RBAC by itself is not sufficient to protect from this. You must isolate the Pods running in your cluster using hypervisor isolated containers or some sort of sandbox, or both.

Every request in Kubernetes is first *authenticated*. Authentication provides the identity of the caller issuing the request. This could be as simple as saying that the request is unauthenticated, or it could intergrade with a pluggable authentication provider (for example Azure Active Directory) to establish an identity within that third party system. Kubernetes does not have a built in identity store; it integrates other identity sources.

Once a user is identified the authorization phase determines if the user is authorized to perform the request. Authorization is a combination of the identity of the user, the resource (effectively the HTTP path), and the verb or action the user is attempting to perform. If user is not authorized a 403 is returned.

### Identity in Kubernetes

Every request that comes to Kubernetes is associated with some identity. Requests with no identity are associated with `system:unauthenticated` group. Kubernetes makes a distinction between user identities and service account identities. Service accounts are created and managed by by Kubernetes itself and are generally associated with components running inside the cluster. User accounts are all other accounts associated with actual users of the cluster (often associated with continuous delivery as a service running outside the cluster).

Kubernetes uses a generic interface for authentication providers. Each of the providers supplies a user name and optionally a set of groups to which the user belongs. Kubernetes supports a number of provides including basic HTTP authentication (deprecated) x509 client certificates, static token files on host, Cloud authentication (Azure AD, AWS IAM etc) and Authentication webhooks. Managed Kubernetes will configure authentication for you, but in your own cluster you get to decide.

### Roles and role bindings

Roles and role bindings are used to determine if a user is authorized to perform an action on an object once the user identity is known. A **role** is a set of capabilities. For example `appdev` role might represent the ability to create Pods and services. A **role binding** is an assignment of a role to one or more identities. Binding the `appdev` role to the user `alice` indicates that Alice has the ability to create Pods and services.

There are two related resources in Kubernetes that represent roles and role bindings. `Role` and `RoleBinding` which applies to just a namespace and `ClusterRole` and `ClusterRoleBinding` which apply across the whole cluster. 

`Role` resources are namespaced, and represent capabilities within that single namespace. You cannot use namespaced roles for non-namespaced resources (e.g., CustomResourceDefinitions), and binding a `RoleBinding` to a role only provides authorization within the Kubernetes namespace that contains both the `Role` and the `RoleDefinition`. 

Below is an example of a role that gives an identity the ability to create and modify Pods and services.

```yaml
kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      namespace: default
      name: pod-and-services
    rules:
    - apiGroups: [""]
      resources: ["pods", "services"]
      verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
```

To bind the above `Role` to the user `alice` we need to create a `RoleBinding` exemplified in the below manifest.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: default
  name: pods-and-services
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: alice
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: mydevs
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-and-services
```

If you want to make the same action but cluster wide you would use `ClusterRole` and `ClusterRoleBinding`. They are largely identical tot he namespaced peers, but have a larger scope.

### Verbs for Kubernetes roles

Commonly used vers in Kubernetes RBAC are listed elow.

| Verb | HTTP method | Description |
| ---- | ----------- | ----------_ |
| create | POST | Create a new resource |
| delete | DELETE | Delete an existing resource |
| get | GET | Get a resource |
| list | GET | List a collection of resources |
| patch | PATCH | Modify an existing resource via a partial change |
| update | PUT | Modify an existing resource via a complete object |
| watch | GET | Watch for streaming updates to a resource |
| proxy | GET | Connect to resource via a streaming WebSocket proxy |

### Using build-in roles

Kubernetes has a large number of well-known system identities (e.g., a scheduler) that require a know set of capabilities. There are also built in cluster roles.  You can view these roles by running the command `kubectl get clusterroles`.  Most of these roles are for system utilities four are intended for generic end users.

* `cluster-admin` role provides complete access to the entire cluster
* `admin` role provides complete access to a complete namespace
* `edit` role allows an end user to modify things in a namespace
* `view` role allows for read only access to a namespace

To see the cluster role binding use the command `kubectl get clusterrolebindings`

#### Auto-reconciliation of built in roles

When the Kubernetes API server starts up, it automatically installs a number of default `ClusterRoles` that are defined int the code of the API server itself. This means if you modify those roles your changes will get discarded when the API server restarts (for example during an update).  To prevent this from happening you need to add the `rbac.authorization.kubernetes.io/autoupdate` annotation with a value of `false` to the built in ClusterRole resource. If this annotation is set to `false` the API server will not overwrite the modified ClusterRole resource.

By default, the Kubernetes API server installs a cluster role that allows `system:unauthenticated` users access to the API server's API discovery endpoint. This is bad for any cluster exposed to a hostile environment such as the open internet. If you need to lock this down, ensure that the `--anonymous=auth=false` flag is set on your API serer.

### Managing RBAC

#### Testing authorization with can-i

You can use the `can i` functionality of `kubectl` to verify if you have access to perform an action. You can also have users use this to validate their permissions.

`kubectl auth can-i create pods`

You can also test sub-resources like logs or port forwarding with the `--subresource` command line flag

`kubectl auth can-i get pods --subresource=logs`

#### Managing RBAC in Source Control

Like all resources in Kubernetes, RBAC resources are modeled using JSON or YAML so they can be source controlled.  `kubectl` has a `reconcile` command that operates similar to `kubectl apply` but for RBAC resources and will reconcile a text-based set of roles and role bindings with the current state of the cluster.

`kubectl auth reconcile -f some-rbac-config.yaml`

If you want to see the changes before they are made you can add the `--dry-run` flag to the command.

### Aggregating ClusterRoles

RBAC supports the usage of an **aggregation rule** to combine multiple roles together in a new role. This new role combined all of the capabilities of all of the aggregate roles together, and any changes to any of the sub-roles will automatically be propagated back into the aggregate role. `ClusterRoles` to be aggregated are specified using label selectors. The `aggregationRule` field in the `ClusterRole` resource contains a `clusterRoleSelector` field, which in turn is a label selector. All `ClusterRole` resources that match this selector are dynamically aggregated into the `rules` array in the aggregate `ClusterRole` resource. 

A best practice for managing `ClusterRole` resource is to create a number of fine-grained cluster roles and then aggregate them together to form higher-level or broadly defined cluster roles. Below is an example of the built in `edit` role defined to be the aggregate of all `ClusterRole` objects that have a label of `rbac.authrization.k8s.io/aggregate-to-edit` set to `true`.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: edit
  ...
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    ...
```

### Using Groups for bindings

When you bind a group to a `ClusterRole` or a namespace Role, anyone who is a member of that group gains access to the resource and verbs defined by that role. To enable any individual go gain access to the group's role, that individual needs to be added to the group. Its better to use groups to control access for obvious reasons. Its hard to manage permissions at an individual person level.

To bind a group to a ClusterRole use a `Group` kind for the `subject` in the binding. In Kubernetes, groups are supplied by authentication providers. There is no strong notion of group in Kubernetes itself; just that an identify can be part of one or more groups, and these groups can be associated with a `Role` or `ClusterRole` via a binding.

```yaml
...
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: my-great-groups-name
    ...
```

## Integrating storage solutions and Kubernetes

### Importing External Services

This section has a side discussion on a use case for namespaces.  In the YAML below you can specify my-database as the database and vary the namespace. That way depending on the namespace you are running in when `my-database` is referenced you will get a different instance. This is good for having a test environment.  The Kubernetes DNS service will return `my-database.test.svc.cluster.internal` in the test namespace and `my-database.prod.svc.cluster.internal` in the prod namespace.

```yaml
kind: Service
metadata:
  name: my-database
  # note 'prod' namespace here
  namespace: prod
...
```

A key difference between a service running in Kubernetes and one that is outside Kubernetes is that in Kubernetes label selectors can be used to identify the dynamic set of Pods what are the backends for a particular service. With a service running outside of Kubernetes this is not the case.

Lets assume we have a service running on the host `database.company.com` To import this external database service into Kubernetes, we start by creating a service without a Pod selector that references the DNS name of the database server.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: external-database
spec:
  type: ExternalName
  externalName: database.company.com
```

When a typical Kubernetes service is created, an IP address is also created and the Kubernetes DNS service is populated with an A record that points to that IP address. When you create a service of type `ExternalName`, the Kubernetes DNS service is instead populated with a CNAME record that points to the external name you specified (`database.company.com` in this example). When an application in the cluster does a DNS lookup for the hostname `external-database.svc.default.cluster`, the DNS protocol aliases that name to `database.company.com`. In this way all containers in Kubernetes believe that they are talking to a service that is backed with other containers.  This technique can also apply to calling cloud services such as cloud provided database which give you a URL for the resource.

If you don't have a DNS name for the external service but just an IP address. The operation to import the service is a little different. You first crate a service without a label selector, but also without a `ExternalName` type we used before. This will make Kubernetes allocate a virtual IP address for this service and populate an A record for it.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: external-ip-database
```

Since there is no selector for the service Kubernetes cannot populate the endpoints for the service for the load balancer to redirect traffic to.  The user is responsible for populating the end-points manually using an `Endpoints` resource as in the example below.  If you have more than one IP address for redundancy, you can repeat them in the `addresses` array. Once the endpoints are populated the load balancer will start redirecting traffic from you Kubernetes service to the IP address endpoints.

```yaml
kind: Endpoints
apiVersion: v1
metadata:
  name: external-ip-database
subsets:
  - addresses:
    - ip: 192.168.0.1
    ports:
    - port: 3306
```

***Note:*** External services do not perform any  health checking.

### Running reliable singletons

The challenge of running storage solutions in Kubernetes is that primitives like ReplicaSet expect that every container is identical and replaceable, but for most storage solutions this is not the case. One option to address this is to run a single Pod that runs the database or other storage solution. This avoids the challenges of running replicated storages in Kubernetes since there is no replication. This is approach is acceptable for environments where limited downtime (for maintenance for example) is acceptable.

#### Running a MySQL singleton

To run a MySQL singleton you will need.

* A persistent volume for the on-disk storage of the application managed independently of the lifespan of the running MySQL application.
* A mySQL Pod that will run the MySQL application
* A service that will expose this Pod to other containers in the cluster

In this example we will use NFS for storage but Kubernetes has many other options which included cloud storage provider options. The YAML example below defines a `PersistentVolume` object which users NFS and has 1GB of space

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: database
  labels:
    volume: my-volume
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  nfs:
    server: 192.168.0.1
    path: "/exports"
```

With the persistent volume created we need to claim that persistent volume for our Pod.  This is done with a `PersistentVolumeClaim` object as in the example below. Note that the `selector` field is used to find the matching volume we defined earlier using a label. You can declare volumes directly inside a Pod specification, but this locks that Pod specification to a particular volume provider. By using volume claims, you can keep your Pod specifications cloud-agnostic; Just create different volumes specific to the cloud, and use a `PersistentVolumeClaim` to bind them together.  

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: database
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  selector:
    matchLabels:
      volume: my-volume
```

Now that we've claimed our volume, we can use a ReplicaSet to construct our singleton Pod. The user or ReplicaSet (instead of just a Pod) but this is better for reliability. Once scheduled to a machine, a bare Pod is bound to that machine forever. If the machine fails, then any Pods that are on that machine that are not being managed by a higher-level controller such as a ReplicaSet vanish with the machine and are not rescheduled elsewhere. For this reason in example below we use a ReplicaSet with a replica size of one for our database.

```yaml
apiVersion: extensions/v1
kind: ReplicaSet
metadata:
name: mysql
  # labels so that we can bind a Service to this Pod
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: database
        image: mysql
        resources:
          requests:
            cpu: 1
            memory: 2Gi
        env:
        # Environment variables are not a best practice for security,
        # but we're using them here for brevity in the example.
        # See Chapter 11 for better options.
        - name: MYSQL_ROOT_PASSWORD
          value: some-password-here
        livenessProbe:
          tcpSocket:
            port: 3306
        ports:
        - containerPort: 3306
        volumeMounts:
          - name: database
            # /var/lib/mysql is where MySQL stores its databases
            mountPath: "/var/lib/mysql"
      volumes:
      - name: database
        persistentVolumeClaim:
          claimName: database
```

The final step is to expose this as a Kubernetes services which is done in the example below. This will expose a service named `mysql` which we can access at the full domain name `mysql.svc.default.cluster`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
ports:
  - port: 3306
    protocol: TCP
  selector:
app: mysql
```

### Dynamic volume provisioning

Many clusters include **dynamic volume provisioning** which has the cluster operator creating one or more `StorageClass` objects. The example below shows a default storage class object that automatically provisions disk objects on the Microsoft Azure platform. Once a storage class has been created for a cluster, you can refer to this refer to this storage class in your persistent volume claim, rather than referring to any specific persistent volume. 

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: default
  annotations:
    storageclass.beta.kubernetes.io/is-default-class: "true"
  labels:
    kubernetes.io/cluster-service: "true"
provisioner: kubernetes.io/azure-disk
```

When the dynamic provisioner sees this storage claim, it uses the appropriate volume driver to create the volume and bind it to your persistent volume claim. The example below shows usage of a `PersistentVolumeClaim` that uses the `default` storage class. The `volume.beta.kubernetes.io/storage-class` annotation is what links this claim back up to the storage class we created. ***Note:*** The lifespan of persistent volumes id dedicated by the reclamation policy of the `PersistentVolumeClaim` and the default is to bind the lifespan of the volume to that of the Pod. This means if you delete a Pod (with a scale down or other event) then the volumes is deleted as well. While this may be the desired behavior you need to be careful not to delete your persistent volumes.

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: my-claim
  annotations:
    volume.beta.kubernetes.io/storage-class: default
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

Persistent volumes are good for traditional applications that need storage, but if you need to develop a high-availability high-availability, scalable storage in a Kubernetes-native fashion you should look into `StatefulSet`. 

[comment]: <> (TODO: Link to StatefulSet once you have that section)

### Kubernetes native storage with StatefulSets

StatefulSets are replicated groups of Pods, similar to ReplicaSets, but unlike RepilcaSets they have the following unique properties.

* Each replica gets a persistent hostname with a unique index (e.g., database-0, database-1, etc.)
* Each replica is created in order from lowest to highest index, and creation will block until the Pod at the previous index is healthy and available. This also applies to scaling up.
* When a StatefulSet is deleted, each of the managed replica Pods is also deleted in order from highest to lowest. This also applied to scaling down the number of replicas.

This set of features makes it easier to deploy storage applications on Kubernetes. For example, the combination of stable hostnames and ordering mean that all replcias, other thant he first one, can reliable reference `database-0` for the purposes of discovery and establishing replication quorum.

#### Manually replicated MongoDB with StatefulSets

[comment]: <> (TODO: This section gets a little messy to follow via my notes. Need to run though this exercise baed on the book and update the notes.)

In this section we set up a replicated MongoDB cluster as an example of using StatefulSets. The example below creates a MongoDB based replicated storage utilizing StatefulSets. When this manifest is deployed with `kubectl apply -f mongo-simple.yaml` you will see the pods coming up one at a time and getting unique names for each Pod.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
spec:
  serviceName: "mongo"
  replicas: 3
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongodb
        image: mongo:3.4.1
        command:
        - mongod
        - --replSet
        - rs0
        ports:
        - containerPort: 27017
name: peer
```

Once the StatefulSet is created, we also need to create a "headless" service to manage the DNS entries for the StatefulSet. In Kubernetes a service is called **headless**  if it doesn't have a cluster virtual IP address. Since with StatefulSets each Pod has a unique identity, it doesn't really make sense to have a load-balancing IP address for the replicated service. You can create a headless service using `clusterIP: None` in the service specification as in the example below.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  ports:
  - port: 27017
    name: peer
  clusterIP: None
  selector:
app: mongo
```

Once the service is created, there are four DNS entries that are populated. `mongo.default.svc.cluster.local` is created, but unlike with a standard service a DNS lookup on this hostname provides all the addresses in the StatefulSet. In addition, entries are created for `mongo-0.mongo.default.svc.cluster.local` and `mongo-1.mongo` and `mongo-2.mongo`. Each of these resolves to the specific IP address of the replica index in the StatefulSet. Thus, with StatefulSets you get well-defined, persistent names for each replica in the set. This useful when you are configuring a replicated storage solution.

At this point you would need to run the below command below in the Pod to set to make `rs0` with `mongo-0.mongo` be the initial primary.

```bash
kubectl exec -it mongo-0 mongo > rs.initiate( {
_id: "rs0",
      members:[ { _id: 0, host: "mongo-0.mongo:27017" } ]
     });
OK
```

To automate the execution of this command we can add an additional containers to our Pods to perform the initialization.  You can use ConfigMap to add a script into the existing MongoDB image as in the example below. Its mounting a ConfigMap volume whose name is `mongo-init`. This ConfigMap holds a script that performs our initialization. It first determines whether it is running on `mongo-0` or not. If it si on `mongo-0` it creates teh ReplicaSet using the same command we ran manually. If its on a different replica it waits until the ReplicaSet exists and then it registers itself as a member of that ReplicaSet.

```yaml
...
- name: init-mongo
  image: mongo:3.4.1
  command:
  - bash
  - /config/init.sh
  volumeMounts:
  - name: config
    mountPath: /config
  volumes:
  - name: config
    configMap:
      name: "mongo-init"
```

The script below sleeps forever after initializing the cluster. Since every container in the Pod needs to have the same `RestartPolicy`. Since we don't want the Mongo cluster to get restarted we need to have our initialization containers run forever as well or Kubernetes may think the Pod is unhealthy.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-init
data:
  init.sh: |
    #!/bin/bash
    # Need to wait for the readiness health check to pass so that the
    # mongo names resolve. This is kind of wonky.
    until ping -c 1 ${HOSTNAME}.mongo; do
      echo "waiting for DNS (${HOSTNAME}.mongo)..."
sleep 2 done
    until /usr/bin/mongo --eval 'printjson(db.serverStatus())'; do
      echo "connecting to local mongo..."
      sleep 2
    done
    echo "connected to local."
HOST=mongo-0.mongo:27017
    until /usr/bin/mongo --host=${HOST} --eval 'printjson(db.serverStatus())'; do
      echo "connecting to remote mongo..."
      sleep 2
    done
    echo "connected to remote."
    if [[ "${HOSTNAME}" != 'mongo-0' ]]; then
      until /usr/bin/mongo --host=${HOST} --eval="printjson(rs.status())" \
            | grep -v "no replset config has been received"; do
        echo "waiting for replication set initialization"
        sleep 2
      done
      echo "adding self to mongo-0"
      /usr/bin/mongo --host=${HOST} \
         --eval="printjson(rs.add('${HOSTNAME}.mongo'))"
    fi
    if [[ "${HOSTNAME}" == 'mongo-0' ]]; then
      echo "initializing replica set"
      /usr/bin/mongo --eval="printjson(rs.initiate(\
          {'_id': 'rs0', 'members': [{'_id': 0, \
           'host': 'mongo-0.mongo:27017'}]}))"
    fi
    echo "initialized"
    while true; do
      sleep 3600
done
```

#### Persistent volumes and StatefulSets

For persistent storage you need to mount a persistent volume in the `/data/db` directory in the Pod template. Because the StatefulSet replicates more than one Pod you cannot simply reference a persistent volume claim. Instead you need to add a **persistent volume claim template**. A claim template is like a Pod template, but it creates volume clmains.

This following needs to be added to your stateful definition. When y ou add a volume claim template to a StatefulSet definition, each time the StatefulSet controller creates a Pod that is part of the StatefulSet it will create a persistent volume claim based on this tempalte as part of that Pod.  For these replicated persistent volumes to work correctly, you either need to have auotprovisions set up for persistent volumes or you need to prepopulate a collection of persistent volume object for the StatefulSet controller to draw from. If there are no claims that can be created, the StatefulSet controlller will not be able to create the corresponding Pods.

```yaml
volumeClaimTemplates:
- metadata:
  name: database
  annotations:
    volume.alpha.kubernetes.io/storage-class: anything
  spec:
    accessModes: [ "ReadWriteOnce" ]
    resources:
    requests:
    storage: 100Gi
```

#### Readines probes for the Mongo cluster

For this example we can use the below for the readines probe.

```yaml
livenessProbe:
  exec:
    command:
    - /usr/bin/mongo
    - --eval
    - db.serverStatus()
  initialDelaySeconds: 10
  timeoutSeconds: 10
```