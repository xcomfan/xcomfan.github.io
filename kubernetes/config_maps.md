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

## Managing ConfigMaps

ConfigMaps are managed through the Kubernetes API. The usual `create`, `delete`, `get` and `describe` commands work for manipulating these objects.

### Listing

`kubectl get configmaps`

`kubectl describe configmap my-config`

### Creating

The easiest way to create a ConfigMap is via `kubectl create configmap`. There are a variety of ways to specify the data items that go into the ConfigMap. These can be combined in a single command.

* `--from-file=<filename>` - Load from the file with the secret data key the same as the filename

* `--from-file=<key>=<filename>` - Load from the file with the secret data key explicitly specified

* `--from-file=<directory>` - Load all files in the specified directory where the filename is an acceptable key name

* `--from-literal=<key>=<value>` - Use the specified key/value pair directly

### Updating

You can update a ConfigMap and have it reflected in running programs. There is no need to restart if the application is configured to reread configuration values. This is a rare feature but might be something you can add to your applications.

Below are three ways to update ConfigMaps or secrets.

#### Update from file

If you have a manifest for your ConfigMap, you can just edit it directly and push a new version with `kubectl replace -f <filename>` You can also use `kubectl apply -f <filename>` if you previously created the resource with kubectl apply. Due to the way that data files are encoded into these objects, updating a configuration can be a bit cumbersome as there is no provision in `kubectl` to load data from an external file. The data must be stored directly in the YAML manifest. The most common use case is when the ConfigMap is defined as part of a directory or list of resources and everything is created and updated together. Oftentimes these manifests will be checked into source control. Be careful not put push secrets into a public location.

#### Recreate and update

If you store the inputs into your ConfigMaps as separate files on disk (as opposed to embedded into YAML directly), you can use `kubectl` to recreate the manifest and then use it to update the object. This will look something like...

[comment]: <> (TODO: I changed the below example from secret to ConfigMap, but its not validated need to check if this works and update with an example I actually tried. I also need to check if this applies to just secrets or if it works for ConfigMaps as well.)

`kubectl create configMap generic kuard-tls --from-file=kuard.crt --from-file=kuard.key --dry-run -o yaml | kubectl replace -f -`

This command line first creates a new ConfigMap with the same name as our existing secret. If we just stopped there, the Kubernetes API server would return an error complaining that we are tying to create a ConfigMap that already exists. Instead, we tell `kubectl` not to actually send the data to the server but instead to dump the YAML that it *would have* sent to the API server to `stdout`. We then pipe that to `kubectl` replace and use `-f -` to tell it to read from `stdin`. In this way we can update a ConfigMap from files on disk without having to manually base64 encode the data.

#### Edit current version

You can use `kubectl edit` to bring up a version of the ConfigMap in your editor so you can modify it.

`kubectl edit configmap my-config`

One you make the desired changes in your editor and save, the new version of the object will be pushed to the Kubernetes API server.

#### Live updates

Once a ConfigMap or secret is updated using the API, it will automatically be pushed to all volumes that use that ConfigMap or secret. This update may take a few seconds. Currently there is no built in way to signal an application when a new version of ConfigMap is deployed. It is up to the application or some helper script to look for the config files to change and reload them.
