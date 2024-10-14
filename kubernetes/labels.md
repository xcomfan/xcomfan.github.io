---
layout: page
title: "Kubernetes Labels"
permalink: /kubernetes/labels
---

## Label Naming and Values

Labels are key/value pairs where both the key and value are represented by strings. Label keys can be broken down into two parts: an optional prefix and a name, separated by a slash. The prefix, if specified must be a DNS sub-domain with a 253 character limit. The key name is required and must be shorter than 63 characters. Names must also start and end with an alphanumeric character and permit the use of dashes `-`, underscores`_` and dots `.`. The contents of the label values follow the same rules as for label keys. Below are some examples of label keys and values

| Key | Value |
| --- | ----- |
| acme.com/app-version | 1.0.0 |
| appVersion | 1.0.0 |
| app.version | 1.0.0 |
| kubernetes.io/cluster-service | true |

## Labels in Kubernetes architecture

Kubernetes is a purposefully decoupled system. There is no hierarchy and all components operate independently. When objects need to be related to each other they are related with labels. For example ReplicaSets which create multiple replicas of a Pod, find the Pods that they are managing via a selector. When you want to restrict network traffic in your cluster, you would use networkPolicy in conjunction with specific labels to identify Pods that should or should not be allowed to communicate with each other. Labels are the glue that holds a Kubernetes application together.

[comment]: <> (TODO: In text above check that networkPolicy is the actual object name)

## Applying and removing labels

To apply or update a label on a running object use the command similar to `kubectl label deployments alpaca-prod "ver=1"` ***Note:*** this command only labels the deployment not the objects which the deployment creates. To change those objects you would need to change the template embedded in the deployment.

[comment]: <> (TODO: Need to test to understand the above statement about deploymet versus object)

You can view labels on an object using the `kubectl describe` command. You can also use the `--show-labels` option in kubectl to see the labels applied to an object. For example `kubectl get pods --show-labels`

To remove a label use a minus at end of label command specifying a key for example `kubectl label deployments alpaca-prod "env-"`. The minus `-` at end of label name removes it.

You may see a label called `pod-template-hash` when viewing labels. This label is applied by the deployment so it can keep track of which Pods were generated from which template version and allows the deployment to manage updates ina clean way.

## Label Selectors

Label selectors are used to filter Kubernetes objects based on a set of labels. Label selectors are used by both end users and by different types of objects (for example how a ReplicaSet relates to its Pods).

You use the `--selector` flag or `-l` for short to use label selectors in kubectl. Below are a few examples.

List pods with `ver` label set to 2 `kubectl get pods --selector="ver=2"`

If you specify two selectors separated by a comma only objects that satisfy both will be returned. This is a logical AND operation: 

`kubectl get pods -l="app=badnicoot,ver-2"`

You can also ask if a label is one of a set of values. `kubectl get pods -l="app in (alpaca,bandicoot)"`

We can also check if a label is set or not. `kubectl get deployments -l="canary"`. This example we are asking for all deployments where the `canary` label is set to anything.

Below is a summary of the Selector operations:

| Operator | Description |
| -------- | ----------- |
| key=value | key is set to value |
| key!=value | key is not set to value |
| key in (value1, value2) | key is one of value1 or value2 |
| key notin (value1, value2) | key is not one of value1 or value2 |
| key | key is set |
| !key | key is not set |

You can combine positive and negative selectors for example `kubectl get pods -l "ver=2,!canary"`
