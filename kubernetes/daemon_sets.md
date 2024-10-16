---
layout: page
title: "Kubernetes DaemonSets"
permalink: /kubernetes/daemon_sets
---

## What are DaemonSets

DaemonSets are a construct that is used to make sure that a Pod is running across a set of nodes in a Kubernetes cluster. DaemonSets are used to deploy daemons such as log collectors and monitoring agents, which typically need to run on every node. You can use labels to run DaemonSet Pods on specific nodes; for example you may want to run special intrusion detection software on nodes that are exposed to the edge network. You can also use DaemonSets to install software on nodes in a cloud-based cluster. If you haver a need to have specific software on every node of your cluster DaemonSets is the way to accomplish this. You can even mount the host filesystem and run scripts that install RPM/DEB packages onto the host operating system. DaemonSets are particularly useful on auto scaled Kubernetes clusters.

## DaemonSet Scheduler

By default, a DaemonSet will create a copy of a Pod on every node unless a node selector is used, which will limit eligible nodes to those with a matching set of labels. DaemonSets determine which node a Pod will run on at Pod creation time by specifying the `nodeName` field in the Pod spec. As a result, Pods created by daemonSets are ignored by the Kubernetes scheduler and are driven by the DaemonSet controller. If a new node is added to the cluster, the DaemonSet controller will notice that it is missing a Pod and will add the Pod to the new node.

## Creating DaemonSets

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

## Limiting DaemonSets to specific nodes

The first step in limiting DaemonSets to specific nodes is to add the desired set of labels to a subset of nodes. This can be achieved using the `kubectl label` command. For example the command below adds the `ssd=true` label to a single node:

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

## Updating a DaemonSet

Prior to Kubernetes 1.6 the only way to update Pods managed by a DaemonSet was to update the DaemonSet and then manually delete each Pod that was managed by the DaemonSet so that it would be recreated with the new configuration. With version 1.6, DaemonSets gained an equivalent to the Deployment object that manages a DaemonSet rollout.

### Rolling update of a DamonSet

Daemon sets can be rolled out using the same `RollingUpdate` strategy that deployments use. You can configure the update strategy using the `spec.updateStategy.type` field, which should have the value `RollingUpdate`. When a DaemonSet has an update strategy of `RollingUpdate`, any change to the `spec.template` field (or subfields) in the DaemonSet will initiate a rolling update.

There are two parameters that control the rolling update of a DaemonSet.

* `spec.minReadySeconds` - determines how long a Pod must be "ready" before the rolling update proceeds to upgrade subsequent Pods. Its good practice to set this to a reasonable long value of 30 to 60 seconds to make sure the Pod comes up healthy.
* `spec.updateStrategy.rollingUpdate.maxUnavailable` - indicates how many Pods may be simultaneously updated by the rolling update. Setting this to 1 is a safe approach as only one node of the cluster at a time will be impacted.

Once a rolling update has started you can use the `kubectl rollout` command to see the current status of a DaemonSet rollout. For example `kubectl rollout status daemonSets my-daemon-set`.

## Deleting a Daemon Set 

`kubectl delete -f fluentd.yaml`

***Note:*** Deleting a DaemonSet will also delete all the Pods being managed by that DaemonSet. You can use the `--cascade=false` option to prevent that from happening.
