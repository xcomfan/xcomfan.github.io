---
layout: page
title: "Kubernetes Role Based Access Control (RBAC)"
permalink: /kubernetes/rbac
---

[comment]: <> (TODO: Need to play around with the Jekyll template to have the Kubernetes section be displaed automatically that way heading can just be the topic and you know what section you are in.)

## What is Role Based Access Control?

Role Based Access Control (RBAC) has been generally available since Kubernetes 1.8. It provides a mechanism for restricting access and actions on Kubernetes APIs to ensure that only appropriate users have access to APIs in the cluster. RBAC is only part of a good security solution. Anyone who can run arbitrary code inside the Kubernetes cluster can effectively obtain root privileges on the entire cluster. RBAC by itself is not sufficient to protect from this. You must isolate the Pods running in your cluster using hypervisor isolated containers or some sort of sandbox, or both.

Every request in Kubernetes is first *authenticated*. Authentication provides the identity of the caller issuing the request. This could be as simple as saying that the request is unauthenticated, or it could intergrade with a pluggable authentication provider (for example Azure Active Directory) to establish an identity within that third party system. Kubernetes does not have a built in identity store; it integrates other identity sources.

Once a user is identified the authorization phase determines if the user is authorized to perform the request. Authorization is a combination of the identity of the user, the resource (effectively the HTTP path), and the verb or action the user is attempting to perform. If user is not authorized a 403 is returned.

## Roles and role bindings

Roles and role bindings are used to determine if a user is authorized to perform an action on an object once the user identity is known. A **role** is a set of capabilities. For example `appdev` role might represent the ability to create Pods and services. A **role binding** is an assignment of a role to one or more identities. Binding the `appdev` role to the user `alice` indicates that Alice has the ability to create Pods and services.

There are two related resources in Kubernetes that represent roles and role bindings. `Role` and `RoleBinding` which applies to just a namespace and `ClusterRole` and `ClusterRoleBinding` which apply across the whole cluster.

`Role` resources are namespaced, and represent capabilities within that single namespace. You cannot use namespaced roles for non-namespaced resources (e.g., CustomResourceDefinitions), and binding a `RoleBinding` to a role only provides authorization within the Kubernetes namespace that contains both the `Role` and the `RoleDefinition`.

Below is an example of a role that gives an identity the ability to create and modify Pods and services.

```yaml
kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      namespace: default
      name: pod-and-services
    rules:
    - apiGroups: [""]
      resources: ["pods", "services"]
      verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
```

To bind the above `Role` to the user `alice` we need to create a `RoleBinding` exemplified in the below manifest.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: default
  name: pods-and-services
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: alice
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: mydevs
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-and-services
```

If you want to make the same action but cluster wide you would use `ClusterRole` and `ClusterRoleBinding`. They are largely identical to the namespaced peers, but have a larger scope.

## Verbs for Kubernetes roles

Commonly used vers in Kubernetes RBAC are listed below.

| Verb | HTTP method | Description |
| ---- | ----------- | ----------- |
| create | POST | Create a new resource |
| delete | DELETE | Delete an existing resource |
| get | GET | Get a resource |
| list | GET | List a collection of resources |
| patch | PATCH | Modify an existing resource via a partial change |
| update | PUT | Modify an existing resource via a complete object |
| watch | GET | Watch for streaming updates to a resource |
| proxy | GET | Connect to resource via a streaming WebSocket proxy |

## Using built-in roles

Kubernetes has a large number of well-known system identities (e.g., a scheduler) that require a known set of capabilities. There are also built in cluster roles. You can view these roles by running the command `kubectl get clusterroles`. Most of these roles are for system utilities four are intended for generic end users.

* `cluster-admin` role provides complete access to the entire cluster

* `admin` role provides complete access to a complete namespace

* `edit` role allows an end user to modify things in a namespace

* `view` role allows for read only access to a namespace

To see the cluster role binding use the command `kubectl get clusterrolebindings`

## Identity in Kubernetes

Every request that comes to Kubernetes is associated with some identity. Requests with no identity are associated with `system:unauthenticated` group. Kubernetes makes a distinction between user identities and service account identities. Service accounts are created and managed by by Kubernetes itself and are generally associated with components running inside the cluster. User accounts are all other accounts associated with actual users of the cluster (often associated with continuous delivery as a service running outside the cluster).

Kubernetes uses a generic interface for authentication providers. Each of the providers supplies a user name and optionally a set of groups to which the user belongs. Kubernetes supports a number of provides including basic HTTP authentication (deprecated) x509 client certificates, static token files on host, Cloud authentication (Azure AD, AWS IAM etc) and Authentication webhooks. Managed Kubernetes will configure authentication for you, but in your own cluster you get to decide.

## Auto-reconciliation of built in roles

When the Kubernetes API server starts up, it automatically installs a number of default `ClusterRoles` that are defined int the code of the API server itself. This means if you modify those roles your changes will get discarded when the API server restarts (for example during an update).  To prevent this from happening you need to add the `rbac.authorization.kubernetes.io/autoupdate` annotation with a value of `false` to the built in ClusterRole resource. If this annotation is set to `false` the API server will not overwrite the modified ClusterRole resource.

By default, the Kubernetes API server installs a cluster role that allows `system:unauthenticated` users access to the API server's API discovery endpoint. This is bad for any cluster exposed to a hostile environment such as the open internet. If you need to lock this down, ensure that the `--anonymous=auth=false` flag is set on your API serer.

## Testing Authorization With can i

You can use the `can i` functionality of `kubectl` to verify if you have access to perform an action. You can also have users use this to validate their permissions.

`kubectl auth can-i create pods`

You can also test sub-resources like logs or port forwarding with the `--subresource` command line flag

`kubectl auth can-i get pods --subresource=logs`

## Managing RBAC in Source Control

Like all resources in Kubernetes, RBAC resources are modeled using JSON or YAML so they can be source controlled.  `kubectl` has a `reconcile` command that operates similar to `kubectl apply` but for RBAC resources and will reconcile a text-based set of roles and role bindings with the current state of the cluster.

`kubectl auth reconcile -f some-rbac-config.yaml`

If you want to see the changes before they are made you can add the `--dry-run` flag to the command.

## Aggregating ClusterRoles

RBAC supports the usage of an **aggregation rule** to combine multiple roles together in a new role. This new role combined all of the capabilities of all of the aggregate roles together, and any changes to any of the sub-roles will automatically be propagated back into the aggregate role. `ClusterRoles` to be aggregated are specified using label selectors. The `aggregationRule` field in the `ClusterRole` resource contains a `clusterRoleSelector` field, which in turn is a label selector. All `ClusterRole` resources that match this selector are dynamically aggregated into the `rules` array in the aggregate `ClusterRole` resource.

A best practice for managing `ClusterRole` resource is to create a number of fine-grained cluster roles and then aggregate them together to form higher-level or broadly defined cluster roles. Below is an example of the built in `edit` role defined to be the aggregate of all `ClusterRole` objects that have a label of `rbac.authorization.k8s.io/aggregate-to-edit` set to `true`.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: edit
  ...
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    ...
```

## Using Groups for bindings

When you bind a group to a `ClusterRole` or a namespace Role, anyone who is a member of that group gains access to the resource and verbs defined by that role. To enable any individual to gain access to the group's role, that individual needs to be added to the group. Its better to use groups to control access for obvious reasons. Its hard to manage permissions at an individual person level.

To bind a group to a ClusterRole use a `Group` kind for the `subject` in the binding. In Kubernetes, groups are supplied by authentication providers. There is no strong notion of group in Kubernetes itself; just that an identify can be part of one or more groups, and these groups can be associated with a `Role` or `ClusterRole` via a binding.

```yaml
...
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: my-great-groups-name
    ...
```
