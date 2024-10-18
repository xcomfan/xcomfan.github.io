---
layout: page
title: "Kubernetes: StatefulSets"
permalink: /kubernetes/stateful_sets
---

## Kubernetes Native Storage with StatefulSets

StatefulSets are replicated groups of Pods, similar to ReplicaSets, but unlike ReplicaSets they have the following unique properties.

* Each replica gets a persistent hostname with a unique index (e.g., database-0, database-1, etc.)

* Each replica is created in order from lowest to highest index, and creation will block until the Pod at the previous index is healthy and available. This also applies to scaling up.

* When a StatefulSet is deleted, each of the managed replica Pods is also deleted in order from highest to lowest. This also applied to scaling down the number of replicas.

This set of features makes it easier to deploy storage applications on Kubernetes. For example, the combination of stable hostnames and ordering mean that all replicas, other thant he first one, can reliably reference `database-0` for the purposes of discovery and establishing replication quorum.

### Manually replicated MongoDB with StatefulSets

[comment]: <> (TODO: This section gets a little messy to follow via my notes. Need to run though this exercise baed on the KUA book and update the notes.)

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

### Persistent Volumes and StatefulSets

For persistent storage you need to mount a persistent volume in the `/data/db` directory in the Pod template. Because the StatefulSet replicates more than one Pod you cannot simply reference a persistent volume claim. Instead you need to add a **persistent volume claim template**. A claim template is like a Pod template, but it creates volume claims.

The following needs to be added to your stateful definition. When you add a volume claim template to a StatefulSet definition, each time the StatefulSet controller creates a Pod that is part of the StatefulSet it will create a persistent volume claim based on this template as part of that Pod. For these replicated persistent volumes to work correctly, you either need to have auto provisions set up for persistent volumes or you need to pre populate a collection of persistent volume objects for the StatefulSet controller to draw from. If there are no claims that can be created, the StatefulSet controller will not be able to create the corresponding Pods.

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

### Readines probes for the Mongo cluster

For this example we can use the below for the readiness probe.

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
