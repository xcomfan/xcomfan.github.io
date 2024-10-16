---
layout: page
title: "Kubernetes Secrets"
permalink: /kubernetes/secrets
---

## What are Secrets in Kubernetes?

Secrets are similar to [ConfigMaps]({% link kubernetes/config_maps.md %}) but focus on making sensitive information available to the workload. They can be used for things like credentials or TLS certificates. Secrets are exposed to Pods via explicit declaration in Pod manifests and the Kubernetes API. In this way, the Kubernetes secrets API provides an application centric mechanism for exposing sensitive configuration information to applications in a way that's easy to audit and leverages native OS isolation primitives.

***Note:*** By default, Kubernetes secrets are stored in plain text in `etcd` storage for the cluster. This may not be sufficient security depending on your requirements. Anyone who has cluster admin rights will be able to read all of the secrets in the cluster. Recent versions of Kubernetes add support for encrypting the secrets with a user supplied key which is generally integrated into a cloud key store. Most cloud key stores have integrations with Kubernetes flexible volumes, enabling you to skip Kubernetes secrets entirely and rely on the cloud providers key store.

## Creating Secrets

Secretes hold one or more data elements as a collection of key/value pairs and are created either with Kubernetes API or the kubectl command-line tool.

Below is an example of using the `kubectl` command to created a secret which stores a TLS certificate and key from files local on the system where the command is being run. The command will create a secret with two elements `kuard.crt` and `kuard.key`

`kubectl create secret generic kuard-tls --from-file=kuard.crt --from-file=kuard.key`

To view the secret use the command below

`kubectl describe secrets kuard-tls`

When creating Secrets keep in mind the [naming constraints]({% link kubernetes/naming_constraints.md %}) for the key names in a Secret. The maximum size for a Secret in Kubernetes is 1MB.

## Consuming Secrets

Secrets can be consumed using the Kubernetes REST API by applications that know how to call that API directly. This approach may not be desirable as this makes your application not portable (will only work in Kubernetes). Another option is to use a **secrets volume**.

### Secrets volumes

Secret data can be exposed to Pods using the secrets volume type. Secret volumes are managed by the `kubelet` and are created at Pod creation time. Secrets are stored on `tmpfs` volumes (aka RAM disks), and as such are not written to disk on nodes. Each element of a secret is stored in a separate file under the target mount point specified in the volume mount.

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

Use the `create secret docker-registry` to create this special kind of secret. For example `kubectl create secret docker-registry my-image-pull-secret --docker-username=<username> --docker-password=<password> --docker-email=<email-address>`

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
