---
layout: page
title: "Kubernetes: Singletons for Persistent Storage"
permalink: /kubernetes/rbac
---

## Running reliable singletons

The challenge of running storage solutions in Kubernetes is that primitives like ReplicaSet expect that every container is identical and replaceable, but for most storage solutions this is not the case. One option to address this is to run a single Pod that runs the database or other storage solution. This avoids the challenges of running replicated storages in Kubernetes since there is no replication. This is approach is acceptable for environments where limited downtime (for maintenance for example) is acceptable.

## Running a MySQL singleton

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

With the persistent volume created we need to claim that persistent volume for our Pod. This is done with a `PersistentVolumeClaim` object as in the example below. Note that the `selector` field is used to find the matching volume we defined earlier using a label. You can declare volumes directly inside a Pod specification, but this locks that Pod specification to a particular volume provider. By using volume claims, you can keep your Pod specifications cloud-agnostic; Just create different volumes specific to the cloud, and use a `PersistentVolumeClaim` to bind them together.

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
