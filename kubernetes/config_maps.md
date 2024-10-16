---
layout: page
title: "Kubernetes ConfigMaps"
permalink: /kubernetes/config_maps
---

## What are ConfigMaps

ConfigMaps are used to provide configuration information for workloads. This information can be a short string or a file. One way to think of ConfigMaps is a Kubernetes object that defines a set ov variables that can be used when defining the environment or command line for your containers.

## Creating ConfigMaps

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

When creating ConfigMaps keep in mind the [naming constraints]({% link kubernetes/naming_constraints.md %}) for the key names in a `ConfigMap`.

As of Kubernetes 1.6 ConfigMaps are unable to store binary data and can only be UTF-8 text. You can in theory encode binary data into a base64 string and put it into the YAML file, but that makes YAML files hard to manage.

Maximum size for a ConfigMap is 1MB.

## Using a ConfigMap

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
