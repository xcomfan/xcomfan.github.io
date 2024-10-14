---
layout: page
title: "Kubernetes Labels and Annotations"
permalink: /kubernetes/labels_and_annotations
---

## What is Kubectl?

`kubectl` is the official Kubernetes client. Its a command line tool for interacting with Kubernetes API. It can be used to manage most Kubernetes objects as well as explore and verity the overall health of the cluster.

## Working With Namespaces and Managing Contexts

By default, kubectl tool interacts with the `default` namespace. If you want to use a different namespace, you can pass the `--namespace` flag for example `kubectl --namespace=mystuff`

If you want to change the default namespace more permanently you can use a **context**. This gets recorded in a kubectl file usually located at `$HOME/.kube/config`.

To create a context with a different default you would run the command `kubectl config set-context my-context --namespace=mystuff`.

This will create a new context but does not actually start using it. To use this context you run the command `kubectl config use-context my-context`.

Contexts can be used to manage different clusters or different users for authenticating to those clusters using the `--users` or `--clusters` flags with the `set-context` command.

## kubectl display formats

You can use the `-o wide` modifier to get more details (in less human readable format that Kubernetes tries to limit to a single line) when using `get` to display resources. You can also use `-o json` to get a JSON output which will have full details. Another common task is extracting specific field from the object.  `kubectl` uses the JSONPATH query language to select fields int eh returned object. For example

`kubectl get pods my-pod -o jsonpath --template={.status.podIP}`

## kubectl describe

If you are interested in details for an object use the command `kubectl describe <resource-name> <obj-name>`. This will provide a rich multi line human readable description of the object as well as related objects and events.

## Checking cluster status

`kubectl version` will display the version of the kubectl tool as well as the version of the server.

`kubectl get componentstatuses` will give you a quick diagnostic for the cluster.

## Listing Kubernetes worker nodes

`kubectl get nodes` lists nodes

`kubectl describe nodes node-1` Give you details about a node hardware, resource utilizations and pods its running.

[comment]: <> (TODO: Below probably needs to be broken up into the section about the specific object.)

## Creating, updating, and destroying Kubernetes objects

Objects in Kubernetes are represented as JSON or YAML files. You can use these YAML or JSON files to create, update or delete object on the Kubernetes server.

If for example you have a simple object stored in `obj.yaml`; you can use `kubectl` to create this object by running the command `kubectl apply -f obj.yaml`. All the details for the object being created will be obtained from the file. if you make changes to the file, you can use the same command to apply those changes. The `apply` tool will only modify objects that are different from the current objects in the cluster. If an object already exists it will simply exit successfully without making any changes.

If you want to see what the `apply` command will do without making the changes you can use the `--dry-run` flag to print the objects to the terminal without sending them to the server.

If you want to make iterative changes instead of editing a local file you can use the command `kubectl edit <resource-name> <obj-name>` which will download the latest object state and launch an editor for you. After you save the file, it will be automatically uploaded back to the Kubernetes cluster.

The `apply` command also records the history of previous configurations in an annotation. You can manipulate these records with the `eit-last-applied`, `set-last-applied` and `view-last-applied` commands. For example `kubectl apply -f myobj.yaml view-last-applied` will show you the last state that was applied to the object. The `-f` option is to specify the file name.

When you want to delete an object use the command `kubectl delete -f obj.yaml`. ***Note:*** `kubectl` will not prompt for a confirmation before deleting an object. There is however a terminating grace period of 30 seconds by default (At least for Pods need to confirm if for other objects).

[comment]: <> (TODO: Do the confrimation bit above)

When a Pod is transitioned to Terminating state it stops taking request. The grace period allows the Pod to finish any active requests that it may be in the middle of processing before it is terminated. When a Pod is deleted any data in the container is deleted as well.If you need to persist data than you need to use `PersistentVolumes`

[comment]: <> (TODO: Above section shuld probably be moved to Pods section in created/deleting.)

To create a deployment in a non declarative way you can use the command `kubectl create deployment <deployment_name> --image=<image> --replicas=<replica_count>` with real values that looks like `kubectl create deployment alpaca-prod --image=gcr.io/kuar-demo/kuard-arm64:blue --replicas=3 --port=8080`