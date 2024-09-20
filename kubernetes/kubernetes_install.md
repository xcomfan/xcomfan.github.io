---
layout: page
title: "Kubernetes Installation"
permalink: /kubernetes/installation
---

## KCA book chapter 2 process

Since the script in the book does not work here is the version that I created based on docs.

```bash
apt-get update
apt-get install -y docker.io
service docker restart
apt-get install -y apt-transport-https ca-certificates curl gnupg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # allow unprivileged APT programs to read this keyring
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' > /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list   # helps tools such as command-not-found to work correctly
apt-get update
apt-get install -y kubeadm kubectl kubelet
```

## Content of the kube.sh script once fixed

kubectl apply -f <https://docs.projectcalico.org/manifests/calico.yaml>
kubectl apply -f <https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml>

## Chapter 2 Lab notes

### Lab 2

### Init cluster

`kubeadm init --pod-network-cidr=192.168.0.0/16`

This gives you the command to run on the second host.

### Kubectl config to point to your cluster

```bash
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

### Add ingress controller and mysterious calico kube controller

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml
```

### Lab 3

use the pv.yaml file in your repo and run the command `kubectl create -f pv.yaml`

Example of sorting the volumes by how much storage capacity they have. `kubectl get pv --sort=.spec.capacity.storage`

### Lab 4 - Performing a Kubernetes upgrade with kubeadm

#### Control Plan Upgrade Steps

Because the book I am using sucks, I need to do some hacking with versions to be able to rproduce the lab so all the commands below are just notes, I did not verify them.

* Upgrade kubeadm on the control plane node.
* Drain the control plane node `kubectl drain ip-172-31-21-95 --ignore-daemonsets`
* Plan the upgrade (kubeadm upgrade plan)
* Apply the upgrade (kubeadm upgrade apply) `sudo apt-get update && sudo apt-get install -y --allow-change-held-packages kubeadm=1.27.2-00 && kubeadm upgrade apply v1.27.2`
* Upgrade kubelet and kubectl on the control plan node `sudo apt-get install -y --allow-change-held-packages kubectl=1.27.2-00 kublet=1.27.2-00 && systemctl daemon-reload && systemctl restart kubelet`
* Uncordon the control plane node `kubectl uncordor ip-172-31-21-95`

#### Worker node upgrade steps

* Drain the node
* Upgrade kubeadm `sudo apt-get install -y --allow-change-held-packages kubeadm=1.27.2-00`
* Upgrade the kubelet configuration (kubeadm upgrade node) `sudo kubeadm upgrade node`
* Upgrade kubelet and kubectl `sudo apt-get install -y --allow-change-held-packages kubectl=1.27.2-00 kubelet=1.27.2-00 && systemctl daemon reload && systemctl restart kubelet`
* Uncordon the node `kubectl uncordon HOSTNAME`

#### Lab 5 - Working with namespaces

Each Kubernetes resource must be in only one namespace and namespaces cannot be nested inside one another.

You can define resource quotas and limits within each namespace to ensure that one tenant cannot monopolize cluster resources. This involves setting CPU, memory and storage limits for each tenant's namespace.

To create a namespace named `dev` use the command `kubectl create namespace dev`

## KCA book chapter 3 notes

For HA in Kubernetes you need to have an multi master cluster.

Kubernetes datastore is, by default, Dqlite, a high availability SQLite.

For HA to work you need to have multiple instances of Kubernetes API running and that means that they need to be behind a load balancer.

**Stacked etcd** If you have an HA setup with control plan running on multiple nodes, then you have etcd running on those nodes and that is stacked etcd. I guess the other option is to have etcd running on other nodes. There is a risk here of loosing both etcd and control plan if a node goes down, but that can be mitigated by adding more hosts.  

**External etcd** This is just etcd running on nodes other than your control plane nodes.

### K8s management tools intro

#### kubeadm

`kubeadm` utility provides `kubeadm init` and `kubeadm join` commands as best-practice quick path ways of establishing Kubernetes clusters.

#### minikube

Lets you set up a cluster using just a single server or machine for practice.

#### helm

Helm provides templating and package management for Kubernetes objects. You can turn your Kubernetes objects or the applications running in your cluster into templates, referred to you by helm as charts to allow you to easily manage complex configurations of multiple Kubernetes objects. You can also download and use shared templates that others have already created.

helm is a package manager that allows you to search, share and use Kubernetes-specific applications. It simplifies the installation and maintenance of Kubernetes apps and is the Kubernetes equivalent of apt or yum.

helm uses a chart packaging format. A chart is a series of files that explain a group of Kubernetes resources. A single chart might be used to deploy anything basic, such as memcaced pod, or something sophisticated, sch as a whole we b app stack with HTTP servers, databases and caches.

#### kompose

Kompose is a tool that can convert docker compose files into kubernetes objects to help you move from local development to running in Kubernetes.

#### kustomize

Kustomize is like helm, but it has a concept of layering where you have a base version and layering yaml artifacts (referred to as patching). This is a strong approach as most firms construct their project from a set of internally developed and off the shelf apps. You can leverage kustomize to absorb any base file modifications for your underlying components while preserving use-case-specific customization overrides.

### Safely Draining a K8s Node

#### Draining

Draining a node will cause the containers running causes containers running on the node to be gracefully terminated and moved to another nodes to prevent service interruption so that you can do maintenance on a node. When a node is being drained Kubernetes will also prevent fresh pods from coming in. If there are deamon set managed pods, the drain does not proceed unless the `--ignore-demonsets` option is used. Unless you use `--force`, the drain does not destroy any pods that do not mirror pods or are handled by a replication controller or are handled by a replicatiosn controller, replica set, DaemonSets, stateful set, or job. If the management resource of one or more pods is absent, `--force` enabled deletion to occur.

Format of the drain command is `kubectl drain <node name>`

When you are ready for the node to operate again use `kubectl uncordon` to make the node schedulable again.

`kubectl drain` by default ignores system pods on the node that cannot be destroyed.

### Upgrading K8s with kubeadm

#### Control plane upgrade steps

Following are the steps to upgrade the control plan node:

* Upgrade kubeadm on the control plane node
* Drain the control plane node
* Plan the upgrade (kubeadm upgrade plan)
* Apply the upgrade (kubeadm upgrade apply)
* Uncordon the control plane node
* Upgrade kubelet and kubectl on the control plane node

#### Worker node upgrade steps (now with more info)

* Upgrade kubeadm
* Drain the node
* Upgrade the kubelet configuration (kubeadm upgrade node)
* Upgrade kubelet and kubectl
* Uncordon the node

#### Recovering from a Failure State

If the kubeadm upgrade fails and does not roll back for example if you lost power or network during execution you can perform it again, the command is idempotent. You can also try `kubeadm upgrade apply --force` to recover from a problematic condition without altering the version of your cluster.

During the upgrading process, kubeadm creates the following backup directories in `/etc/kubernetes/tmp`

* `kubeadm-backup-etcd-<date>-<time>` - backup of this control plane nodes local etcd member data. If upgrade fails contents of this folder can be manually restored in /var/lib/etcd.
* `kubeadm-backup-manifests-<date>-<time>` - copies this control plane node's static Pod manifest files. If an upgrade fails and the automated reollback also fails, the contents of this folder can be manually restored in `/etc/kubernetes/manifests`.

### Overview of upgrading K8s with Kubeadm

#### Steps/commands to upgrade a control plane node

Command to drain control plane node looks like `kubectl drain k8s-control --ignore-daemonsets`

Command to execute the given update kubeadm looks like `sudo apt-get && sudo apt-get install -y --allow-change-held-packages kubeadm=1.22.2-00` (this is from the crappy book real command is just similar)

To check kubeadm version use `kubeadm version`

To "plan the upgrade" use the command `sudo kubeadm upgrade plan v1.22.2`

To upgrade the control plan components the command looks like `sudo kubeadm upgrade apply v1.22.2`

To upgrade kubelet and kubectl on the control plan node use the command `sudo apt-get update && sudo apt-get install -y --allow-change-held-packages kubelet=1.22.2-00 kubectl=1.22.2-00`

To reload the daemon use the command `sudo systemctl daemon-reload`

To restart kubelet use the command `sudo systemctl restart kubelet`

To uncordon the control plane node use the command `kubectl uncordon k8s-control`

To verify the control plane is working use `kubectl get nodes`

#### Steps/commands to upgrade worker node

To drain worker node use the command `kubectl drain k8s-worker1 --ignore-daemonsets --force`

To upgrade kubeadm on the worker node `sudo apt-get update && sudo apt-get install -y --allow-change-held=packages kubeadm=1.22.2-00`

To upgrade the kubelet configuration on the worker node `sudo kubeadm upgrade node`

To upgrade kubelet and kubectl on the worker node use `sudo apt-get update && sudo apt-get install -y --allow-change-held-packages kubelet=1.22.2-00 kubectl=1.22.2-00`

To reload the daemon on worker node `sudo systemctl daemon-reload`

To restart kubelet on worker node 1 `sudo systemctl restart kubelet`

To uncordon the worker node `kubectl uncordon k8s-worker1`

To verify the worker node is working `kubectl get nodes`

### Backing up and restoring the etcd cluster data

Some noteworthy features of etcd are

* Fully replicated - Each node in an etcd cluster has full access to the datastore
* Highly Available - etcd is intended to have no single point of failure and endures hardware failures and network partitions gracefully
* Reliably Consistent - Each data 'read' returns the most recent write from all clusters
* Fast - The benchmark for etcd is 10,000 writes per second
* Secure - etcd supports automated Transport Layer Security (TLS) and optional SSL client certificate authentication. Its a best practice to use least priviledge to etcd as etcd maintains critical and highly sensitive configuration data.
* Simple - Using standard HTTP/JSON tools, any application may read or publish data to etcd

#### Backing up and restoring etcd

Etcd is where all Kubernetes object are kept. Backing up the etcd cluster data is critical for recovering Kubernetes cluster in catastrophe scenarios, such as losing all control plane nodes. All Kubernetes states and crucial information is contained int he snapshot file. Encrypt the snapshot files to keep important Kubernetes data safe.

There are two ways to back up an etcd cluster, built-in snapshots and volume snapshots.

##### Built in Snapshot

You can backup etcd data using the etcd command-line tool `etcdctl` and its snapshot save command.  Example of running a backup below...

```bash
$ ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT
snapshot save <file name>
```

another example below because the book I am using is trash.

`ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT snapshot save snapshotdb`

To verity the snapshot use `ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshotdb`

The snapshot would be taken on a live member. Taking the backup does not impact the members performance.

When using etcdctl there are some options to be aware of.  Below is an example of providing a an endpoint certificate.

`ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacerts=<trusted-ca-file> --cert=<cert-file> --key=<key-file> snapshot save <backup-file-location>`

To restore etcd the command would look like `ETCDCTL_API=3 etcdctl snaphsot restore <file-name>` There are options you can add to this command one of which has the restore creating a new logical cluster. The snapshot file could be from a backup or from a directory that still exits.

If the access URLs of the restored cluster differ from those of the prior cluster. In that case, the Kubernetes API server must be adjusted. You would do that by restarting the API server with the `--etcd-servers=$NEW_ETCD_CLUSTER $OLD_ETCD_CLUSTER` (replace the variables with the new and old cluster IPs). If a load balancer is in front of etcd cluster that may need to be adjusted as well.

A full example of the backup command is below which has hints on where to get certificates from.

```bash
export ETCDCTL_API=3
etcdctl snapshot save --ednpoints=127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
/home/ubuntu/myk8scluster2.db
```

To get your nodes information go to the manifests directory look for the `etcd.yaml` file in `/etc/kubernetes/manifests/` that will have all the info you need for parameters in the backup command.

If its not obvious what restoring etcd would do: If for example you took an etcd backup and proceeded to delete a pod that was running when backup was taken.  When you restore the deleted pod would come back.

##### Volume Snapshot

This approach backs up etc data by taking a snapshot of the storage volume operating on a storage volume that supports backup such as Amazon EBS.

## KCA book chapter 4 notes (Kubernetes object management)

### Overview of kubectl

kubectl looks for a file named `config` in the `$HOME/.kube` directory for configuration. You can use the `KUBECONFIG` environment variable or the `kubeconfig` flag to spacify different config files.

Syntax of the kubectl command is `kubectl [command] [TYPE] [NAME] [flags]` where...

* command - defines the action you want to take on one or more resources, such as create, get, describe, or delete full list in table below.
* TYPE - Indicates the type of resource. You can provide singular or plural or shortened forms of resource types, which are case-insensitive.
* NAME - Specifies the resource's name. Case matters. If the name is missing all resources of the type are displayed.
* flags - Optional flags for the command. Flags will override any default values and environment variables.

kubectl commands
[comment]: <> (TODO: The syntax column is not very usable need to review and improve)

| Operation | Syntax | Description |
| --------- | ------ | ----------- |
| create | `kubectl create -f FILENAME [flags]` | Create one or more resources from a file or stdin |
| delete | `kubectl delete (-f FILENAME or TYPE or NAME or NAME or -l label or --all) [flags]` | Delete resources either from file, or stdin, or specifying label selectors, names resource selectors, or resources |
| describe | `kubectl describe (-f FILENAME or TYPE  [NAME_PREFIX or NAME or -l label]) [flags]` | Display the detailed state of one or more resources |
| exec | `kubectl exec POD [-c CONTAINER] [-i] [-t] [flags] [--COMMANDS] [args]` | Execute a command against a container in a pod |
| get | `kubectl get (-f FILENAME or TYPE or [NAME or /NAME or -l label]) [--watch][--sort-by=FIELD][[-o or --output]=OUTPUT_FORMAT][FLAGS]` | List one or more resources |
| apply | `kubectl apply -f FILENAME [flags]` | Apply a configuration change to a resource from a file or stdin.

### Lab: exploring a Kubernetes Cluster with kubectl

To get a capacity sorted list of persistent volumes use the command: `kubectl get pv --sort-by=.spec.capacity.storage`

To run a command inside the "Quark Pod's" container use `kubectl exec quark -n beebox-mobile -- cat /etc/key/key.txt`

To create a deployment using a spec file use `kubectl apply -f /home/deployment.yml`

### Kubectl tips

Breakdown of kubernetes management techniques:

You get undefined behavior if you mix these techniques. Pick one and use that.

| Management technique | Description | Operates on | Recommended environment | Supported writers | Notes |
| -------------------- | ----------- | ----------- | ----------------------- | ----------------- | ----- |
| Imperative commands | User operates directly on live items in a cluster | Live objects | Development projects | 1+ | Does not provide history of past configurations because it works on live objects |
| Imperative object configuration | DESC | Individual files | Production projects | 1 | |
| Declarative object configuration | DESC | Directories of files | Production projects | 1+ | |

The `--record` flag can be used to write the command that was run in the kubernetes.io/change-cause resource annotation. The documented change can be used as a reference point in the future. For example, it can be used to look at the commands that were run in each Deployment revision.

### Managing K8s role based access control (RBAC)

RBAC is a Kubernetes feature that allows you to control who has access to what in the cluster.

There are 4 types of Kubernetes objects declared via the RBAC API.

* Role
* ClusterRole
* RoleBinding
* ClusterRoleBinding

### Role and ClusterRole

An RBAC Role is a collection of rules that reflect a set of permissions. Permission are only used in conjunction (there are no deny rules). A Role always sets permissions within a specific namespace thus when creating a role you must specify the namespace to which it belongs.

A ClusterRole is a nameless resources. Because a Kubernetes object can only be namespaced or not namespaced the resources Role and ClusterRole have different names.  ClusterRoles can be used to ...

* Permissions on namespaced resources can be defined and granted within each namespace
* Permissions on named resources can be defined and granted across all namespaces
* Permissions on cluster-scoped resources can bed defined.

TLDR: Use a Role to define a role within a namespace and a ClusterRole to define a role that spans the entire cluster.

#### Role example

Below is an example of a role that can be used to allow read access to pods in the "default" namespace

```yaml
apiVersion:rbac.authorization.k8s.io/v1
kind: role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
```

#### Cluster role example

Cluster role can grant same permissions as a role but it has cluster scope.  Below are a few examples of permissions that make sense as cluster scope.

* Resources with a cluster scope such as nodes
* Endpoints that are not resources such as `/healthz`
* Spanning all namespaces, namespaced resources such as Pods

For example a ClusterRole can allow a certain user to perform kubectl obtain pods --all-namespaces.

Below is an example of a ClusterRole used to allow read access to secrets across all namespaces.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
    #
    # at the HTTP level, the name of the resource for accessing Secret
    # objects is "secrets"
    resources: ["secrets"]
    verbs: ["get", "watch", "list"]
```

### Role binding and ClusterRole bindin

A role binding assigns a user or group of users the permissions indicated in a role. It contains a list of topics (users, groups, or service accounts) and a reference to the role that is being assigned. A valid path segment name must be used as the name of a roleBinding or ClusterRoleBinding object.

#### Role binding examples

Within the "default" namespace, below is an example of a roleBinding that grants the "pod-reader" Role to the user "jane".

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
    name: jane # "name" is case sensitive
    apiGroup: rbac.authorization.k8s.io
roleRef:
  # 'roleRef' specifies the binding to a Role / ClusterRole
  kind: Role # this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

A ClusterRole can be referenced by a RoleBinding to provide the rights described in that ClusterRole to resources inside the RoleBinding's namespace. This type of reference enables you to define a set of shared roles for your cluster and then reuse them across multiple namespaces.

#### ClusterRoleBinding example

A `ClusterRoleBinding` can be used to provide permissions across an entire cluster. Any user in the group `manager` can read secrets in any namespace using the `ClusterRoleBinding` below.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
    name: manager # Name is case sensitive
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

You cannot change the Role or ClusterRole that a binding refers to once created. You will get a validation error if you try to alter the roleRef of a binding. If you want to update a binding's roleRef, you must first remove the binding object and construct a replacement.

### Creating a service account

Kubernetes service accounts are resources generated and managed using the Kubernetes API. They are used to authenticate in-cluster Kubernetes-produced entities like Pods to the Kubernetes API server or external services.

A service account in Kubernetes is an account that container processes in pods use to authenticate with the Kubernetes API. If your pods need to communicate with the API, you can restrict their access via service accounts. Service accounts use role-based access control objects for access control, just like any other user account. Therefor we can use ClusterRoles or ClusterRoleBindings to bind service accounts to give Kubernetes API functionality to our pods that use those service accounts.

### Inspecting Pod Resource Usage

#### kubectl top

The kubectl top command shows current CPU and memory usage for all pods or nodes in a cluster or a single pod or node if one is specified.  For example to see resource utilization across nodes use the command `kubectl top node`

Another good example of kubectl top usage: `kubectl top pod -n beebox-mobile --sort-by-cpu --selector app=auth`

#### Kubernetes metrics server

You can install kubernetes metric server with the command `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/metrics-server-helm-chart-3.8.2/components.yaml%20-0%20metric-server-components.yaml` (THe address here is probably wrong).  You can then validate that the metric server is running with the command `kubectl get --raw /apis/metrics.k8s.io/`

## KCA Book chapter 5 notes (pods and container)

### Kubernetes Pods

A Pod is the lowest deployment and management unit in Kubernetes. 

### Managing Application Configuration

You will probably at some point want to pass dynamic values to your applications at runtime to control how they behave. This process is known as application configuration, where you are just passing data to containers that control what they do and how they run.

1. A file that defines the configuration for a Kubernetes object is known as an object **configuration file** or just **configuration file**. Configuration files are often kep in source control system like Git.

2. Live object configuration or **live configuration** is the values of an objects live configuration as monitored by the Kubernetes cluster. These are often saved in the Kubernetes cluster storage `etcd`.

3. A declarative configuration writer or declarative writer is a human or software component that modified a live object.

### ConfigMaps

A **ConfigMap** is simply a Kubernetes object representing a key-value store of configuration data. If you have configuration that you need to pass into your pods and your containers it is often a good idea to store those configuration in a ConfigMap that would offer you a centralized place to store your configuration data within the Kubernetes cluster. Once you have that key-value data inside your ConfigMap, you can pass it into your containers so they can use it. If you need to update those configurations, it is just a matter of updating the ConfigMap and potentially restarting any pods using that data.

There are two primary ways to store configuration data in Kubernetes: ConfigMap and Secrets. Secrets are very similar to ConfigMpas, but they are designed to store sensitive data securely. 

### Environment Variables

You can pass ConfigMap and secret data to containers using environment variables and those variables are visible at runtime to the container process. In the containers spec, you define an environment variable that you pull from a particular key within a ConfigMap, which will be passed as a container process environment variable. To set Kubernetes environment variables you may utilize the `env` or `envFrom` fields. 

### Configuration Volumes

The other way to pass data to containers is through configuration volumes. Within the configuration volume, configuration data from ConfigMaps or Secret is passed in the form of a mounted volume. It means that configuration data appear in files available on the container file system. Specifically, you have a file for each top-level key in the configuration data, and the contents of that file are going to be all of the data and sub-keys associated with that top-level key. To utilize a volume, describe where to mount those volumes into containers `in.spec.containers` and specify the volumes to supply for the Pod `in.spec.volumes[*]` Volumes are mounted within the image at the provided paths.

below is an example ConfigMay specification. You can create this using the command `kubectl create -f my-configmap.yml` and display it using `kubectl describe configmap my-configmap`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
data:
  key1: Hello, world!
  key2: |
    Test
    multiple lines
    more lines
```

below is an example of a Secret specification. You need to store the secret in base 64 encryption which you can generate with `echo -n 'secret' | base64`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  secretkey1: base_64_value_goes_here
  secretkey2: another_base_64_value_here
```

Below is an example of secrets  being used in a Pod configuration naturally you would create this pod with the command `kubectl create -f env-pod.yml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'echo "configmap: $CONFIGMAPVAR secret: $SECRETVAR"']
    env:
    - name: CONFIGMAPVAR
      valueFrom:
        configMapKeyRef:
          name: my-configmap
          key: key1
    - name: SECRETVAR
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: secretkey1
```

Below is an example of exposing data using configuration volumes. You an create this pod with `kubectl create -f volume-pod.yml` and execute it with `kubectl exec volume-pod -- ls /etc/config/configmap` or `kubectl exec volume-pod -- cat /etc/config/configmap/key1` (you get the idea)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    volumeMounts:
    - name: configmap-volume
      mountPath: /etc/config/configmap
    - name: secret-volume
      mountPath: /etc/config/secret
  volumes:
  - name: configmap-volume
    configMap:
    name: my-configmap
  - name: secret-volume
    secret:
      secretName: my-secret
```

### Managing container resources

Kubernetes has the ability to specify the resource requirements of each container. A containers memory and CPU requirements and resource limits can be defined in the spec. 

#### Resource Requests

When you specify a resource request for a Pod's containers, the kube-scheduler utilized this information to choose which node to deploy the Pod on. When you establish a resource limit for a container, the kubelet enforces those limitations preventing the running container from using more of that resource than the limit you specified. the kubelet additionally reserve the requested quantity of that system resource for that container's exclusive usage. It is important to note the containers can use more than the requested resources. The resource requests do not force the container to stay within that limit, as resource requests only affect scheduling. Resource requests allow you to specify the number of necessary resources to run a container. They govern which worker nodes the containers should be scheduled on. When Kubernetes is getting ready to run a particular pod, it chooses a worker node based on the resource requests of that Pod's containers. Kubernetes uses those values to choose a node with enough resources to run that Pod. If the Pod node has enough resources available, a container may (and is permitted) utilize more resources than its requested for that resource specifies. On the other hand, a container cannot exceed its resource limit.

Below is an example of setting resource limits in a spec.  Memory is measured in bytes, and CPU is measured in CPU units which are 1/1000 of one CPU.  In example below 250m or milli-CPUs is being requested which equates to 25% of one CPU.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: busybox
    image: busybox
    resources:
      requests:
        cpu: "250m"
        memory: "128Mi"
```

#### Resource Limits

Resource limits provide us with a way to limit the amount of resources containers can use and stop containers from using more resource thant hey should. It is important to note that the container runtime is responsible for enforcing resource limits when you use resource limits. Different containers runtimes do this in different ways; for example, some runtimes might enforce resource limits by terminating the container process.If you are using resource limits your containers may get stopped if they attempt to use more resources then specified in the limit.

Resource limits allow you to put some constraints on how many resources you container can use and to prevent certain containers from consuming a bunch of resources and running away with all the resources in your cluster, potentially causing issues for other containers and applications. If you specify a 4GiB memory limit ofr that container, the kubelet (and container runtime) enforces it. The container is prevented from exceeding the set resource limit by the runtime. For example, suppose a process in the container attempts to use more memory than is permitted. In that case, the system kernel end the process with an Out-Of-Memory (OOM) error.

Below is an example of what resource limits look like in a container spec.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  -name: busybox
  image: busybox
  resources:
    limits:
      cpu: "250m"
      memory: "128Mi"
```

### Monitoring Container Health with Probes

#### Container health

Kubernetes has the ability to restart unhealthy containers. To make this work Kubernetes needs to accurately determine the status of applications.

#### Pod lifecycle

Pods have a specified lifetime that begins with the **Pending** phase and progresses to the **Running** phase if at least one of its primary containers starts normally. then comes the **Succeeded** or **Failed** phases, depending on whether any container in the Pod failed.

While a Pod is operation, the kubelet can restart containers to manage various issues. Kubernetes tracks different container statuses within a Pod and determines the action to to take to restore the Pod's health. Pods have a specification and a real status in the Kubernetes API. A Pod object's state comprises a set of Pod conditions. If your application requires it, you may integrate specific readiness information into the condition data. Pods can only be scheduled once in their lives. When a Pod is scheduled (assigned) to a Node, it runs on that Node until it is stopped or terminated.

#### Pod lifetime

Kubernetes uses a higher-level abstraction known as a **controller** to manage the relatively disposable Pod instances. Pods are generated, given a Unique ID (UID), and allocated to nodes, where they remain until tey are terminated (according to the restart policy) or deleted. A given Pod (as specified by a UID) is never "rescheduled" to a different node. Instead, Pod can be replaced by a new and nearly identical one with a different UID. When a volume is said to have the same lifetime as a Pod it means that the object lives for as long as that precise Pod (with the exact UID). If that Pod is removed for any reason, the linked item (volume in this case) is destroyed and recreated.

#### Container states

Kubernetes maintains the status of each container within a Pod in addition to the overall phase of the Pod. Con-tainer lifecycle hooks can trigger events at certain stages in a containers's lifespan. Once a Pod is assigned to a node by the scheduler, the kubelet begins building containers for that Pod using a container runtime. Containers can be in one of three states: waiting, Running or Terminated. Each state has a distinct meaning.

* Waiting - A container in the waiting state is still doing the actions required for a successful startup, such as retrieving the container image from a container image registry. Kubectl will shwo you the reason a container is in waiting state when you `kubectl describe pod pod-name`

* Running - This status means container is running normally. If a postStart hook was configured, it was al-ready run and completed.

* Terminated - This state means the container either completed or failed for some reason. When use use kubectl to query a Pod with a terminated container, you will see a reason, an exit code, and the start and end times for the container's runtime duration. If a preStop hook is configured on a container, it is executed before the container reaches terminated state.

#### Container Probes

A probe may be used to inspect a container using one of the following mechanisms. A probe is a diagnostic done on a container regularly by Kubernetes.

* Exec - Inside the container, it executes a specific command. If the command exists with a status code of 0, the diagnostic is considered successful

* gRPC - Uses gRPC to make a remote procedure call, and gRPC health checks should be implemented by the target. If the status of the answer is SERVING, the diagnostic is regarded as successful. gRPC probes are an alpha feature that may be accessed only by enabling the gRPC ContainerProbe feature gate.

* httpGet - Executes an HTTP GET request on a given port and route against the Pod's IP address. The diagnostic is considered successful if the answer is more than or equals 200 but less than 400.

* tcpSocket - Performs a TCP check on a given port against the Pod's IP address. If the port is open, the diagnostic is considered successful. It is considered healthy if the distant system (the container) instantly shuts the connection once it opens.

#### Types of probes

On a running container, the kubelet can optionally execute and respond to three types of probes:

* LivenessProbe - Liveness probes allow us to automatically determine whether or ot a container application is in a healthy state. By default, Kubernetes only considers a container to be down or broken if the container process stops, but liveness probes can allow us to customize this detection mechanism and make it more sophisticated. A container could be running, and the process has not stopped, but things are still broken. With liveness probes you can detect more sophisticated and subtle situations where things are not working, but the container is still running.

* StartupProbe - These are very similar to liveness probes, but they constantly run on a schedule. Startup probes only run at container startup and stop once they succeed. Essentially, startup probes detect when the application has successfully started up. They monitor container health during hte startup process. Startup probes are useful for legacy applications with long startup times. If a StartupProbe is used, all other probes are blocked until the StartupProbe succeeds. If a startup probe fails, the kubelet terminates the container and subjects it to the restart policy.

* ReadinessProbe - Applications are occasionally unavailable to serve traffic for a short period of time. For example, during startup, an application may need to load massive data or configuration files, or it may need to rely on external services. You do not want to destroy the application in this situation, but you also do not want to send it requests. To detect and mitigate these issues, Kubernetes provides readiness probes. A Pod with container reporting that they are not ready does not receive traffic.

### Building self-healing Pods with Restart Policies

#### Restart Policies

There are 3 possible restart policies in Kubernetes: Always, OnFailure, and Never. The default setting is Always and the restart policy is specified in the Pod spec. All containers in the Pod are affected by the restartPolicy.

[comment]: <> (TODO: Because the book is garbage when I see it somewhere else need to get a better explainer of what happens if a container runs to completion with this policy.)
**Always** Wit this policy containers are always restarted if they stop or become unhealthy. This policy should be used for an application that should essentially just always be running.

**OnFailure** If you have some software designed to run once to successful completion and then stop, and it does not need to be run again you want to use the OnFailure policy. This policy automatically restarts an application if it fails, but will not be restarted if it succeeds or finishes.

**Never** With this restart policy no matter what happens, if a container completes successfully, it will never be restarted. It would be best to use this for applications that only run once and never restart automatically.

### Creating Multi Container Pods

#### Multi Container Pod

In a multi-container pod, the containers share resources such as network and storage that can interact, working together to provide functionality. 

#### Cross Container Interaction

Once you have multiple containers in the same pod, how do they interact with each other? Containers in the same Pod can communicate with each other using shared resources. One such shared resources is the network so containers in the same Pod can communicate with each other on any port event if that port is not exported to the cluster. They can also share storage and use same volumes to share data within a Pod.

#### Shared Volumes in Kubernetes Pod

A shared volume is a siple and effective way to transfer data amongst containers in a Pod in Kubernetes. Data on Kubernetes Volumes can survive container restarts, but theses volumes have the same lifetime as the Pod. 

#### Inter-Container Network Communication

Containers in a Pod can be accessing using "localhost" and share the same network namespace. In addition, for containers the observable hostname is the name of the Pod. Because containers share the same IP address and port space you need to coordinate port usage for apps within a Pod.

#### Inter Process Communication (IPC)

Containers in a Pod share the same IPC namespace thus they are able to interact via normal inter-process communication such as SystemV semaphores or POSIX shared memory.

#### Multi-Container Pods Use Case

### Introduction to Init Containers

An init container in Kubernetes is the one that starts and executes before other containers in the same Pod. Its purpose is to carry out initialization logic for the primary application hosted on the Pod. Create the appropriate user credentials, make database migrations, and design schemas. You can have more than one init containers in a Pod. Before the next init container can begin, the previous one must finish successfully. If a Pod's init container fails the Pod wil be continuously restarted till it succeeds unless you have a restart policy of Never in which case the init container failing will fail the whole Pod. If you provide several init containers for a Pod, kubelet executes each init container in the order specified before the next init container may execute the previous one must succeed. 

#### How Init Containers Differ From Regular Containers

All app container fields and features, including resource limitations, volumes, and security settings, are supported by init containers. The resource demands and limits for an init container, on the other hand, are handled differently. In addition because they must run to completion before a Pod may start, init containers do not support lifecycle, livenessProbe, readinessProbe, or startupProbe.

#### Detailed behavior of init containers

Kubelet pauses launching init containers at Pod startup until networking and storage are available. All init containers must execute again if a Pod is restarted. 

#### Use cases for init containers

* Pause a pod to wait for another service to become available. For example you can use an init container to pause startup until a service or another pod becomes available.
* Perform sensitive startups steps (such as loading secrets/certificates) outside of app containers. 
* Populate data into a shared volume at startup.
* Communicate with another external service for example if you need to register Pod with some external service.

#### The yaml spec for an init containers

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
  initContainers:
  - name: delay
    image: busybox
    command: ['sleep', '30']
```

## Chapter 6: Advanced Pod Allocation

### Intro

The kube-scheduler in Kubernetes is in charge of scheduling pods to specific nodes in the cluster. By default nodes are picked based on each container's resource demands and restrictions in the produced Pod. While scheduling pods based on resource limits works in many cases sometimes you want to assign ods to specific nodes. This is where advanced pod scheduling comes in.

## Exploring K8s Scheduling

### Scheduling

The kube-scheduler does the scheduling. The scheduler does the scheduling based on 1. whether or not the resources requests specified in a pod can be fulfilled by the available resources in a node. 2. Different configuration that affect the label of the node and nodeSelector specified in a pod to select a suitable node for it.

#### nodeSelector

nodeSelector is a field in the pod descriptor file to specify the node(s) on which it is to be run. The node's label is used in the nodeSelector field to select and schedule a specif node.

```yaml
apiVersion: v1
kind: Pod
metadata:
  nodeSelector:
    special: "true"
  containers:
  - name: nginx
    image: nginx:1.19.1
```

#### nodeName

If you want to bypass regular scheduling mechanisms you can it by using the name of a node rathern thatn the label of that node.  Under spec in the pod descriptor file, nodeName is the field used to specify the node's name to select.

## Using DaemonSets

A **DaemonSet** is an object runs a copy of a pod on each node automatically. When new node are added to the cluster, pod copies are added. If you delete a DaemonSet, all the copies of pods that DaemonSet created will also be deleted.

Example of a DaemonSet descriptor file

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-daemonset
spec:
  selector:
    matchLabels:
      app: my-daemonset
  template:
    metadata:
      labels:
        app: my-daemonset
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.1
```

In a simple case, one DaemonSet could be used for each type of daemon, covering all nodes. In a comlex case, multiple DaemonSets could be sued for a single type of daemon with different minimum requirements of the CPU and memory usage for different types of hardware.

### Advanced Scheduling and Pod Affinity and Anti Affinity

You can establish rules for how pods should be arranged in relations to other pods using pod affinity and anti affinity. Custom labels on nodes and label selectors supplied in pods are used to define the rules.

## Using Static Pods

Static pods are used when you want tot urn them on a node without the involvement of the control plane. You cannot manage these static pods through the k8s API server. These static pods are managed by kubelet instead of by the K9s API server.

To set up a static pod...

1. Log into a worker node
2. Create the manifest file in the location `/etc/kubernetes/manifests/beebox-diagnostic.yml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: beebox-diagnostic
spec:
  containers:
  - name: beebox-diagnostic
    image: acgorg/beebox-diagnostic:1
    ports:
    - containerPort: 80
```

3. Start the static pod by restarting kubelet to get it to pick up the static config file. `sudo systemctl restart kubelet` Once that is done you should be able to see the status of the static pod via the `kubectl get pods` command. The static pod you see when you run `kubectl get pods` is just a mirror of the static pod and if you try to `kubectl delete pod beebox-diagnostic-k8s-worker1` it will just re-appear because the API has no control over it.

## Chapter 7 Deployments

### K8s Deployments Overview

#### What is a Deployment?

A Deployment is a Kubernetes object described in a yaml descriptor file. It is used to create replicas of a pod. For example you would use a deployment when you wish to create five pods with similar specifications. You do not need to create each pod individually. Deployment is used to define the desired state for replicas of the pods. The deployment controller changes the original state to the desired state and maintains the desired state by deleting, creating, and replacing pods with new configurations.

Example of descriptor file for a deployment.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-deployment
  template:
    metadata:
      labels:
        app: my-deployment
  spec:
    containers:
    - name: nginx
      image: nginx:1.19.1
      ports:
      - containerPort: 80
```

#### Desired State

The number of replicas of the pods, a selector, and a template are all considered the desired state.

* **Replicas** - Number of pod replicas managed by the deployment

* **Selector** - A selector used to identify the replica pods based on their labels, to be managed by the deployment

* **Template** - A pod definition template used to create replica pods

#### Benefits of Kubernetes Deployment

Since a Kubernetes deployment controller constantly monitors the status of pods and nodes, it can replace a failing pod or bypass down nodes, ensuring that vital applications continue to run.

Deployments have several use cases, such as:

1. An applications can easily be scaled up or down by increasing or decreasing the number of replicas.
2. Rolling updates can be performed by deploying a newer version of the application.
3. An application acn be rolled back to a previous version
4. A new state of the pods can be declared by updating the specifications of the pod template. Pods are moved from the old replicaSet to the new replicaSet at a controlled rate by deployment.
5. The rollout of a deployment can be paused to make multiple changes or fixes to its template specifications and then resumed after that.
6. Find out if a rollout has stuck by using the deployment status(progressing, complete, or fail to progress as an indicator)
7. Old replicaSets that are not needed anymore can be removed.

#### Deployment Strategies

##### Recreate Deployment

The recreate strategy 'recreates' existing pod instances by terminating them and replacing them with the new versions. This is most commonly utilized in dev environments where user impact is not an issue. Recreate completely refreshes the pods and the applications state. As a result, both the shutdown of the old deployment and the recommencement of instances of the new deployment cause downtime.

##### Rolling Update Deployment

ling update allows for a smooth gradual transition from one application version to the next. A new ReplicaSet containing the new version is launched, and replicas of the old version are terminated as replicas of the new version are launched. All of the previous version pods are eventually decommissioned and replaced by the new version.

##### Blue/Green Deployment

The Blue/Green strategy allows for a quick transition from old to new version once the new version has been tested. The new 'green' version is used alongside the previous 'blue' version. When the 'green' version is confirmed to be working as intended, the version label in the selector field of the Kubernetes Service object that handles load balancing is replaced. This automatically redirects traffic to the latest version. Although this method allows for a quick rollout while avoiding versioning concerns, it consumes twice the resources because both versions are active until cut-over.

##### Canary Deployment

A smaller group of users is routed to the new version of an application operating on a smaller subset of pods to test functionality in a production environment in a canary deployment. Once testing has been completed with out errors, clones of the new version are scaled up, and the old version is gradually phased out. Canary deployments are useful for testing new functionality on a small sample of users and can be easily undone.

#### Rollback

All the deployment versions are kept in Kubernetes so when needed they can be rolled back. Only the template specification part of the deployment can be updated or degraded as a rolling update or rollback. Any change made to this part creates a new revision or a new deployment version. 

[comment]: <> (TODO: Sentance above int he book does not make sense, I need to verify)

## Chapter 8 Networking

### K8s Networking Architectural Overview

#### Pods

Kubernetes uses an "IP-per-pod" architecture, in which each pod is given its IP address, and all containers in a pod share the same network namespace and IP address. The IP address of each Pod is unique to the cluster. The Pod is a group of containers that share the same node's networking and storage resources. Each Pod has its own IP address, and all of the containers in the Pod share storage, IP addresses, and port space (network namespace).  All containers in a Pod can reach each others localhost port and thus that all container in a pod must coordinate port usage.

#### Containers

Containers networking allows containers to communicate with other containers, a host, and external networks.

#### Nodes

Using a Pod's IP address, any Pod can contact another Pod. This builds a virtual network that allows Pods to connect no matter which node they are on. In this way the Kubernetes network makes nodes completely transparent. For a pod to communicate with another Pod, there is no need to know what node its running on.

With the exception of intentional network segmentation regulations Kubernetes imposes the following criteria on any networking implementation.

1. Pods on a node can connect with all pods on all nodes without the use of NAT agents. For example, system daemons and kubelets can communicate with all pods on a node.

2. Without NAT, pods in a node's host network can communicate with pods on all other nodes.

### How does networking in Kubernetes work?

Because in Kubernetes the internal network is insulated from the external network, one of the issues of Kubernetes networking is dealing with how internal (east-west) and external (north-south) traffic interact. External traffic can be injected into a Kubernetes cluster in a number of different ways.

#### LoadBalancer

In this scenario redirects all external traffic to a service and every service is assigned a unique IP address.

#### ClusterIP

The default Kubernetes service for internal communication is ClusterIP. External traffic, on the other hand, can use a proxy to connect to the default Kubernetes ClusterIP service. This is important for troubleshooting services and viewing internal dashboards.

#### NodePort

NodPort is a service that opens ports on nodes or virtual machines and forwards traffic form the ports to the service. This is mostly used for services that do not need to be online all of the time such as demos.

#### Ingress

Ingress is a load balancer that functions as a router or controller, routing traffic to services. It is useful if you wish to provide various services using the same IP address.

### CNI Plugins Overview

Kubernetes network plugins are knows as **CNI plugins**. These plugins offer network communication between pods by the Kubernetes network model's specification.

#### Selecting a Network Plugin

CNI plugins are compliant with the Container Network Interface (CNI) specification, designed to ensure interoperability. The CNI specification v0.4.0 is followed by Kubernetes.

**kubenet plugin** uses the bridge and host-local CNI plugins to implement basic cbr0.

Kubernetes nodes will stay in the NotReady state until a network plugin is implemented. As a result, unless you install that network plugin, you cannot run ods in your cluster because none of the nodes will be available, and any pods you try to create will remain pending, waiting for a node to become ready.

#### Installing a Network Plugin

The kubelet comes with a single default network plugin and a cluster-wide default network. when it first starts up, it looks for plugins, remembers what it finds, and runs the selected plugin at the relevant points in the pod's lifespan (this is only true for Docker, as CRI manages its own CNI plugins). When utilizing kubelet plugins, there are two command-line parameters to remember.

1. cni-bin-dir - On initialization, kubelet looks for plugins in this directory.
2. network plugin - The cni-bin-dir network plugin to utilize. It has to be the same name as a plugin from the plugin directory that has been probed.

### Understanding K8s DNS

Kube-DNS and CoreDNS are two well known DNS solutions for configuring DNS naming rules and mapping pod and service DNS to cluster IP addresses. Kubernetes services can be identified by a name that corresponds to any number of backend pods maintained by the service. DNS has a consistent naming convention, making the addresses of multiple services easier to remember. Services can be referred to not just by their Fully Qualified Domain Name (FQDN) but also by their own names.

#### How does Kubernetes DNS work?

You may build up a DNS system in Kubernetes using two well-supported add-ons: Core DNS and Kube-DNS. CoreDNS is a recent add-on that, as of Kubernetes v1.12, has become the default DNS server.

Kubernetes DNS schedules a DNS Pod and Service on the cluster, then configures the kubelets to notify individual containers to resolve DNS names using the DNS Service's IP. A DNS name is assigned to each service defined in the cluster (including the DNS server itself)

The DNS system assigns domain and subdomain names to pods, ports and services, allowing the components in your Kubernetes cluster to find them. DNS-based service discovery is extremely powerful as you do not have to hard-code network information like IPs and ports into your application.

#### Service DNS Record

A records, CNAME records, and SRV records are all supported by Kubernetes.

##### A Records

The most basic sort of DNS record is an A record which is used to point a domain or subdomain to a certain IP address. The domain name, the IP address used to resolve it, and the TTL in seconds are all included in the record.

##### CNAME

A domain or subdomain can point to another hostname using CNAME records. CNAME accomplish this by using the existing A record as their value. An A record, on the other hand, resolves to a specific IP address. CNAME records can also be utilized for cross-cluster service discovery with federated services in Kubernetes. There is a common service across several Kubernetes clusters in this instance. This service is discoverable by all pods, regardless of which cluster they are in. Such an approach enabled cross-cluster services discovery, which is a large topic in and of itself.

##### SRV Records

By specifying the protocol(s) and address of some services, SRV records aid service discovery. An SRV record defines the priority, weight, port, and target for a given service adn the symbolic name and transport protocol (e.g, TCP) used as part of the domain name.

#### Pod's DNS Records

##### A Records (pod)

If DNS is enabled, pods are given a DNS address. Pod-ip-address.my-namespace.pod.cluster.local is a record type. An entry of form 172-12-3-4.default.pod.cluster.local would be created for a pod with IP 172.12.3.4 in the namespace default and a DNS name of cluster.local.

##### Pods Hostname and Subdomain Field

The metadata.name value of a pod determines its default hostname. The default hostname can be changed by entering a new value in the optional hostname box. In the subdomain box, users can also choose a custom subdomain name. For example, a pod in namespace my-namespace with the hostname custom-host and the subdomain custom-subdomain will have the Fully Qualified Domain Name custom-host.custom-subdomain.my-namespace.svc.cluster.local.

### NetworkPolicies

A network policy is an object that lets you manage the flow of network traffic to and from your pods. This enabled you to create a more secure cluster network by isolating pods from the traffic they do not require.

#### Pod Selector

The pod selector specifies which pods in the namespace are affected by the network policy. The pod selector can use labels to choose pods.  All pods in a cluster are isolated by default

#### A network policy can apply to ingress traffic, egress traffic, or both types of traffic

Ingress traffic is network traffic that enters the pod from a source outside the pod, whereas egress traffic is network traffic that leaves the pod for another location. Therefore, network policies can regulate traffic entering a pod, traffic leaving a pod, or both. The network policy determines which traffic is allowed via the from and to selectors related to entrance and egress.

#### From and To selectors

##### From Selector

Ingress traffic will be allowed if it is selected from the selector. It determines which inbound traffic is permitted to enter the pod.

##### To Selector

For the outgoing egress traffic, a "to selector" achieves the same thing. It chooses which outgoing traffic will be allowed. Depending on on whether the policy applies to egress, ingress or both, we will find our from selectors in the ingress section and our to selectors in the egress section of the network policy definition. There are multiple selector types that are available for to and from.

1. We can use a pod selector to choose pods based on their labels. if we use it in conjunction with an ingress rule in our from selector, pods with that app equal db label will be able to communicate with the network policy affected pods.

2. We can choose a namespace based on labels, and then the network policy will allow any traffic coming from or going to pods within those namespaces.

3. The ipBlock selector can be used to choose an IP range using CIDR notation. As a result, an IP range is chosen to allow incoming and outgoing traffic.

#### Limiting Resource Usage

The number and capacity of resources available to a namespace are limited by resource quota. This is most commonly used to limit how much CPU, RAM or persistent disk a namespace can allocate, but it can also be used to limit how many pods, services, or volumes each namespace can have.

#### Network access restrictions

Application writers can use a namespace's network policies to limit which pods from other namespaces can access pods and ports within their namespaces. Many of the Kubernetes networking providers that are supported now adhere to network policies.

Quota and limit ranges can also be used to govern whether users can request node ports or load-balanced services, which can affect whether those users' applications are visible outside of the cluster on many clusters.

Additional precautions such as per-node firewalls, physically separating cluster nodes to avoid cross-talk, or advanced networking policies, may be available that manage network rules on a per-plugin or per-environment basis.

#### Restricting Cloud Metadata API access

Metadata services are frequently exposed locally to instances by cloud platforms (AWS, Azure, GCE and so on). By default, pods running on an instance can access these APIs, which can contain cloud credentials for that node or provisioning data like kubelet credentials. This account's credentials can be sued to elevate within the cluster or to other cloud services. Limit the permissions given to instance credentials when running Kubernetes on a cloud platform, use network policies to restrict pod access to the metadata API, and avoid using provisioning data to deliver secrets.

#### Default Deny

You can use the default deny network policy to block all traffic except for what is needed to avoid potential malicious traffic. Default deny network blocks all traffic by default unless we explicitly allow network communication.

Example of a default deny policy below and you can apply it with the command `kubectl create -f default-deny-all-np.yml`

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: nptest
spec:
  podSelector: {}
  policyType:
  - Ingress
  - Egress
```

## Chapter 09: Services

