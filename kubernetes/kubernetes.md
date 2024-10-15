---
layout: page
title: "Kubernetes"
permalink: /kubernetes
---

[comment]: <> (TODO: Should put the Docker stuff into its own docker section)

[comment]: <> (TODO: Need to organize this monolith)

[comment]: <> (TODO: Need to try this and write up a very basic example The book uses kuard as example but that is too comlex for a reference.)

[comment]: <> (TODO: Below is Alpahbetical for now, but should be more nested and by logical concept order to make it easy to navigate At the moment it just a tracker of all the pages to force all links to work.)

[Annotations]({% link kubernetes/annotations.md %})

[Cluster Components]({% link kubernetes/cluster_components.md %})

[Deployments]({% link kubernetes/deployments.md %})

[Ingress]({% link kubernetes/ingress.md %})

[Labels]({% link kubernetes/labels.md %})

[Namespaces]({% link kubernetes/namespaces.md %})

[Nodes]({% link kubernetes/nodes.md %})

[Pods]({% link kubernetes/pods.md %})

[ReplicaSets]({% link kubernetes/replica_sets.md%})

[Troubleshooting]({% link kubernetes/troubleshooting.md %})

[Volumes]({% link kubernetes/volumes.md %})

------------------------------------------------

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

## Extending Kubernetes

Even cluster admins should be careful and use diligence when extending Kuberneetes with third party tools. Some extensions can be used as a vector to steal secrets or run malicious code. Extending a cluster makes it different than stock Kubernetes. When running multiple clusters, it is very valuable to build tooling to maintain consistency of experience across clusters which shold include extensions that are installed.

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

At this point we can list the loadtest resource but it does not do anything yet. You canuse the CRUD API to create read update delete objects but they will take no action. To actually do something a controller needs to be present in the cluster to react to the new API we defined. We need a piece of code that will continuously monitor the custom resources and create, modify, or delete LoadTests as necessary to implement the API. There is a Watch API on the API server that your code can use to implement this control loop. The API is a bit hard to use so you should look into supported mehcanisms such as `informer` in the [client-go library](https://pkg.go.dev/k8s.io/client-go/informers)


[comment]: <> (TODO: I stopped outlining this chapter as it became kind of har to follow without working though the examples. Will re pass later)

### Patterns for custom resources

#### Just data

In the Just data pattern, you are simply using the API server for storage and retrieval of information for your application. You should not use the API server for storage and retrieval for your application; its snot designed for that. API extensions should be used for control or configuration object that help you manage the deployment or runtime of your application. An examle use case for "just data" might be configuration for canary deployments of your application. While you can use ConfigMaps; ConfigMaps are untyped and sometimes using a more strongly types API extension object provides clarity and ease of use. Extensiosn that are just data don't need a corresponding controller to activate them, but they may have validating or mutating admission controlles to ensure that they are well formed.

#### Compilers

This pattern knows as "compiler" or "abstraction" pattern has the API extension object representing a higher-level abstraction that is "compiled" into a combination of lower level Kubernetes objects. The LoadTest extension we discussed prior is an example of this pattern. A user consumes the extension as a high level concept, in this examle a `loadtest`, but it comes into being by being deployed as a collection of Kubernetes Pods and services. To achive this, a compiled abstraction requries an API controller to be running somewehre in the cluster, to watch the current LoadTests and create the "compiled" represntation. There is however no online helath maintenances.

#### Operators

Operator pattern provides online proactive management of the resources created by the extension. These extensions likely provide a higher level abstraction (for example a database) that is compield down to a lower level representation, but they also provide online functionality such as snapshot, backups, or upgrades. To achieve this, the controller not only monitors the running state of the application supplied by the extension to add or remove things as necessary, but also monitors the running state of the application supplied by the extension and takes actions to remediate unhealthy databases, take snapshots or restore from a snapshot if failure occurs. Operators are the most complicated pattern for API extension. 

#### Getting started

The [Kubebuilder project](https://kubebuilder.io) is a good resource to use when getting started with extension development.

## Organizing your application

### Principles to guide us

* Filesystems as the source of truth
* Code review to ensure the quality of changes
* Feature flags for staged roll forward and roll back

#### Feature gate and guards

Feature gates and guards play an important role in bridging the gap of developing new features in source control and the deployment of those features into production. You can do the development behind a feature flag or gate as in the sippet below. This enables commiting of code to the production branch long before the feature is ready to ship. The enabling of the feature requries just a configuration change to activate the flag. This makes it very clear what chagned in the production environment, and makes it easy to roll back without having to go back to an older version of the code.
 
```javascript
if (featureFlags.myFlag){
  // Feature implementation goes here
}
```

#### Filesystem layout

The first cardinality on which you want to organize your application is the semantic component or layer (e.g., frontend, batch work queue etc.) This may seem like overkill when one team manages all these components but it makes it easy to scale the team later. An application with a front end that uses two services would have a file layour similar to the below. Within each directory the configuration for each application is stored. These are YAML files that directly represent the current state of the cluster. Its generally useful to include both service namea nd the object type withing the same file. While Kubernetes allows for the creation of YAML files with multiple objects in the same file, this should generally be avoided. The only good reason to group objects in the same file is if they are conceptually identical. If grouping objects togetehr doesn't form a single concept, they probably shouldn't be in a single file.

```text
frontend/
  frontend-deployment.yaml
  frontend-service.yaml
  frontend-ingress.yaml
service-1/
  service-1-deployment.yaml
  service-1-service.yaml
  service-1-configmap.yaml
service-2/
  ...
```

#### Managing periodic versions

It is very useful to be able to look back historically and see what your application deployment previously looked like. It is also useful to be able to iterate a configuration forward while still being able to deploy a stable release configuration. Thus its handy to be able to simultaneously store and maintain multiple different revisions of your configuration. There are two approaches to this.

The first is to use tags, branches and source-control features. This leads to a more simplified directory structure and aligns with how source code revisions are managed. The second options is to clone the configuration within the filesystem and use directoreis for different revisions. This approach is convinient because it makes simultaneous viewing of the configuration very straightforward. Both of these are essentially identical and are just a matter of preference.

##### Versioning with branches and tags

When you are ready for realease, you place a source-control tag (e.g `git tag v1.0`) on the config version and the HEAD continues to iterate forward. When you need to update the release configuration; first you commit the change to HEAD of the repository, then you create a new branch named v1 at the v1.0 tag. You then cherry-pick the desired change onto the release branch (`git cherry-pick <edit>`), and finally you tag this branch with the `v1.1` tag to indicate the new point release. ***Note:*** A common error is to cherry pick fixes into a release branch only. Its a good idea to cherry-pick it into all active releases, in case for some reason you need to roll back versions but the fix is still needed.

##### Versioning with directories

In this approach your versioned deployment exists within its own directory as in the example below. All deployments occur from `HEAD` instead of from specific revision tags. When adding a new configuration it is done to the files in the `current` directory. When creating a new rlease the current directory is copied to create a new directory associated wit the new release. When performing a bugfix change to a release, the pull request must modify the YAML file in all the relevant release directories. This is a slightly better experience than the cherry-picking approach described earlier, since it is clear in a single change request that all of the relevant versions are bing updated with the same change, instead of requiring a cherry-pick per version.

```text
frontend/
  v1/
    frontend-deployment.yaml
    frontend-service.yaml
  current/
    frontend-deployment.yaml
    frontend-service.yaml
service-1/
  v1/
    service-1-deployment.yaml
    service-1-service.yaml
  v2/
    service-1-deployment.yaml
    service-1-service.yaml
  current/
    service-1-deployment.yaml
    service-1-service.yaml
...
```



### Structuring your appliation for development, testing and deployment

There are two goals for your application with regard to development and testing.

* Each developer should be able to easily develop new features for the application. Developers should be able to work in their own environment with all services available.
* You should be able to easily and accurately test your application prior to deployment. This is essential to being able to quickly roll out features while maintaining availability.

#### Progression of a release

To achieve both these goals it is important to relate stages of development to the release versions described earlier. The stages of release are:

* HEAD - The bleeding edge of the configuration; the latest changes
* Development - Largely stable, but not ready for deployment. Suitable for developers to use for building features.
* Staging - The beginning of testing, unlikely to change unless problems are found.
* Canary - The first realease to users, used to test for problems with real world traffic and likewise give user a chance to test what is coming next.
* Release - the current production release

You should use a tag to mark the development stage. An automated process should be used to test the `HEAD` branch, and if test pass the `development` tag is moved to `HEAD`. Thus developers can track reasonably close to the latest changes when deploying their own environments.

To map the stage to a specific revision if using source control approach you would use tags and if using file system approach you would use symbolic links.

#### Parameterizing your applications with templates

Since its impractical to keep the development stages identical, but you want the environments to be as identical as possible its a good idea to have parametrirized environments with templates. This lets you use a template for the bulk of the configuration but have a limited set of parameters to produce the final configuration. This makes it easy to see the differences between environments.

##### Parameterizing with Helm and templates

[Helm](https://helm.sh) is a package manager for Kubernetes. They patterns described here for Helm will apply to whatever templating option you choose.

Helm templating language use the "mustache" syntax, so for exmple

```yaml
metadata:
  name: {{ .Release.Name }}-deployment
```

indicates that `Release.Name` should be substituded into the name of a deployment. To pass a parameter for this value you use `values.yaml` file with contents like the below.

```yaml
Release:
  Name: my-release
```

After paramter substitution you would get ...

```yaml
metadata:
  name: my-release-deployment
```

##### Filesystem layour for parameterization

With parametarization, instead of treating each deployment lifecycle stage as a pointer to a version, each deployment lifecycle is the combination of a parameter file and a pointer to a specific version.  Below is an example of such a layout.  

```text
frontend/
  staging/
    templates -> ../v2
    staging-parameters.yaml
  production/
    templates -> ../v1
    production-parameters.yaml
  v1/
    frontend-deployment.yaml
    frontend-service.yaml
  v2/
    frontend-deployment.yaml
    frontend-service.yaml
```

Doing this with version control looks similar, except that the parameters for each life-cycle stage are kept at the root of the configuration directory tree.

```text
frontend/
  staging-parameters.yaml
  templates/
  frontend-deployment.YAML
```

Should you need to have different configurations for different regions, your directory layout would lool like the below.

```text
frontend/
  staging/
    templates -> ../v3/
    parameters.yaml
  eastus/
    templates -> ../v1/
    parameters.yaml
  westus/
    templates -> ../v2/
    parameters.yaml
...
```

If using version control your layout would look like...

```text
frontend/
  staging-parameters.yaml
  eastus-parameters.yaml
  westus-parameters.yaml
  templates/
    frontend-deployment.yaml
...
```
## System containers vs application containers

System containers seek to mimic virtual machines and often run a full boot process. They often include a set of system services typically found on a VM, such as ssh, cron and syslog. Over time this approach became seen as poor practice and application containers gained favor. Application containers commonly run a single problem. While a single program per container may seem like an unnecessary constrains, it provides the perfect level of granularity for composing scalable applications and is a design philosophy heavily used by Pods.