---
layout: page
title: "Kubernetes: Dynamic Volume Provisioning"
permalink: /kubernetes/dynamic_volume_provisioning
---

Many clusters include **dynamic volume provisioning** which has the cluster operator creating one or more `StorageClass` objects. The example below shows a default storage class object that automatically provisions disk objects on the Microsoft Azure platform. Once a storage class has been created for a cluster, you can refer to this storage class in your persistent volume claim, rather than referring to any specific persistent volume.

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

When the dynamic provisioner sees this storage claim, it uses the appropriate volume driver to create the volume and bind it to your persistent volume claim. The example below shows usage of a `PersistentVolumeClaim` that uses the `default` storage class. The `volume.beta.kubernetes.io/storage-class` annotation is what links this claim back up to the storage class we created. ***Note:*** The lifespan of persistent volumes is dictated by the reclamation policy of the `PersistentVolumeClaim` and the default is to bind the lifespan of the volume to that of the Pod. This means if you delete a Pod (with a scale down or other event) then the volumes is deleted as well. While this may be the desired behavior you need to be careful not to delete your persistent volumes.

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

Persistent volumes are good for traditional applications that need storage, but if you need to develop a high-availability high-availability, scalable storage in a Kubernetes-native fashion you should look into [StatefulSet]({% link kubernetes/stateful_sets.md %}).
