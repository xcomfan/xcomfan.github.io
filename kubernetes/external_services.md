---
layout: page
title: "Kubernetes: Importing External Services"
permalink: /kubernetes/external_services
---

In the YAML below you can specify my-database as the database and vary the namespace. That way depending on the namespace you are running in when `my-database` is referenced you will get a different instance. This is good for having a test environment. The Kubernetes DNS service will return `my-database.test.svc.cluster.internal` in the test namespace and `my-database.prod.svc.cluster.internal` in the prod namespace.

```yaml
kind: Service
metadata:
  name: my-database
  # note 'prod' namespace here
  namespace: prod
...
```

A key difference between a service running in Kubernetes and one that is outside Kubernetes is that in Kubernetes label selectors can be used to identify the dynamic set of Pods that are the backends for a particular service. With a service running outside of Kubernetes this is not the case.

Lets assume we have a service running on the host `database.company.com` To import this external database service into Kubernetes, we start by creating a service without a Pod selector that references the DNS name of the database server.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: external-database
spec:
  type: ExternalName
  externalName: database.company.com
```

When a typical Kubernetes service is created, an IP address is also created and the Kubernetes DNS service is populated with an A record that points to that IP address. When you create a service of type `ExternalName`, the Kubernetes DNS service is instead populated with a CNAME record that points to the external name you specified (`database.company.com` in this example). When an application in the cluster does a DNS lookup for the hostname `external-database.svc.default.cluster`, the DNS protocol aliases that name to `database.company.com`. In this way all containers in Kubernetes believe that they are talking to a service that is backed with other containers. This technique can also apply to calling cloud services such as cloud provided database which give you a URL for the resource.

If you don't have a DNS name for the external service but just an IP address. The operation to import the service is a little different. You first crate a service without a label selector, but also without a `ExternalName` type we used before. This will make Kubernetes allocate a virtual IP address for this service and populate an A record for it.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: external-ip-database
```

Since there is no selector for the service Kubernetes cannot populate the endpoints for the service for the load balancer to redirect traffic to. The user is responsible for populating the end-points manually using an `Endpoints` resource as in the example below. If you have more than one IP address for redundancy, you can repeat them in the `addresses` array. Once the endpoints are populated the load balancer will start redirecting traffic from you Kubernetes service to the IP address endpoints.

```yaml
kind: Endpoints
apiVersion: v1
metadata:
  name: external-ip-database
subsets:
  - addresses:
    - ip: 192.168.0.1
    ports:
    - port: 3306
```

***Note:*** External services do not perform any health checking.