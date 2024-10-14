---
layout: page
title: "Troubleshooting"
permalink: /kubernetes/troubleshooting
---

## Debugging Commands

To see the logs for a running container use `kubectl logs <pod-name>`. If you have multiple containers in your Pod you can specify the container with the `-c` flag. You can also use the `-f` flag to stream the logs.

To execute commands in a running container use the command `kubectl exec -it <pod-name> -- bash`. This will provide you with an interactive shell inside the running container so that you can perform more debugging. If you don't have bash or another terminal you can always attach to the running process using `kubectl attach -it <pod-name>`. This will attach to the running process and allow you to send input to the running process (assuming process is set up to read from stdin)

You can copy files to and from a container using the cp command: `kubectl cp <pod-name>:</path/to/remote/file> </path/to/local/file>`. Generally speaking copying files to a pod is an anti patterns, but it may be useful when putting out a fire.

If you want to access your Pod via the network, you can use the `port-forward` commands to forward network traffic from the local machine to the Pod. This enables you to securely tunnel network traffic through to containers that might not be exposed anywhere on the public network. For example the following command opens up a connection that forwards traffic from the local machine on port 8080 to the remote container on port 80.

`kubectl port-forward <pod-name> 8080:80`

You can also use the `port-forward` command with services by specifying `services/<service-name>` instead of `<pod-name>`, but note that the requests for a service will be forwarded to a single Pod, not go though the service load balancer.

If you are interested in how your cluster is using resources you can use `kubectl top nodes` or `kubectl top pods` to see the pods running and their resource utilization.

## Tools and resources

* The [kind](https://kind.sigs.k8s.io) project lets you run Kubernetes using containers as nodes so you can play around in your local environment.

* There are plugins for VSCode, IntelliJ, Eclipse etc for working with your cluster.

* `kubectl` supports integration with your shell to enable tab completion for both commands and resources. You may need to install the `bash-completion` package to make this work.

* For log aggregators look into `fluentd` and `elasticsearch`.

## Labels and Annotations

*Labels* are key value pairs that can be attached to Kubernetes objects such as Pods and ReplicaSets. They are useful for attaching identifying information to Kubernetes objects. Labels provide the foundation for grouping objects. *Annotations*, on the other hand, provide a storage mechanism that resembles labels. Annotations are key value pairs designed to hold non-identifying information that can be leveraged by tools and libraries.