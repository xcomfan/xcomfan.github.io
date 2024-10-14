---
layout: page
title: "Service Discovery"
permalink: /kubernetes/service_discovery
---

[comment]: <> (TODO: Right now this section is a bit of a catch all. I don't have enough experience with the topics yet to logically break this up, but this very much needs to be broken up as I work though examples. I also flattened the headings so need to reorder those in a logical manner. Right now most headings are ## level.)

## What is Service Discovery

Kubernetes is a dynamic system where Pods are placed on nodes, and the numbers of Pods running can vary based on load. This makes it easy to run a lot of things but you also need to be able to find those running things. This class of problem is referred to as **service discovery**. Service discovery tools help solve the problem of finding which processes are listening at which addresses for which services.

## The Service object

In Kubernetes service discovery starts with a **Service object**. A Service object is a way to create a named label selector, but it has some other functionality.

To imperatively create a Service object use the command similar to `kubectl expose deployment alpaca-prod` (assuming you have the alpaca-prod) deployment already running. The `kubectl expose` command will pull both the label selector and the relevant ports from the deployment definition to set up the service. It will also assign a virtual IP (called **cluster IP**) to the service. This is a special IP address which Kubernetes will use to load balance across all the Pods that are identified by the selector. This process of having Pods that match a selector get load balanced in the cluster IP is the service discovery mechanism (just my take away need to confirm)

You can get a list of services with the command `kubectl get services`

## Service DNS

Because the cluster IP is virtual, it is stable and it is appropriate to give it a DNS address. Kubernetes provides a DNS service exposed to Pods running in the cluster and provides DNS names to cluster IPs.

An example of the DNS name and how it breaks down is `alpaca-prod.default.svc.cluster.local` where...

* `alpaca-prod` is the name of the service in questions
* `default` is the namespace that the service is in
* `svc` is for recognizing that this is a service
* `cluster.local` is the base domain name for the cluster

When referring to a service in your own namespace you can just use the service name. You can also refer to a service in another namespace by having the namespace in the specified name (`alpaca-prod.default` for example) or you can use the FQDN.

## Readiness Checks

One nice things the Service object does is track if a Pod is ready to accept requests or not. This is useful if your applications needs some time to initialize when it starts up. This is the readiness check functionality. You can specify the readiness check in your manifest file with code similar to

```yaml
speck:
  ...
  template:
  ...
  spec:
    containers:
      ...
      name: alpaca-prod
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        periodSeconds: 2
        initialDelaySeconds: 0
        failureThreshold: 3
        successThreshold: 1
```

In the example above the readiness check will look for successful GET request to `/ready` endpoint on port 8080. It will check every 2 seconds starting as soon as the Pod comes up. If three successive checks fail then the Pod will be considered not ready. If one checks succeeds the Pod will again be considered ready. Only ready pods are sent traffic. This functionality is not just useful at startup its also a good way for an overloaded or sick server to signal to the system that it doesn't want to receive traffic anymore, and is a good way to implement a graceful shutdown. Server can signal it no longer wants to receive traffic complete all jobs or connections and shut down.

## Exposing a service outside a cluster

You can use the `NodePort` functionality to have your service be accessible from any cluster node. Essentially if you specify `spec.type` as `NodePort` or use `--type=NodePort` when calling expose command Kubernetes will assign a port to your service that can be used from any cluster node to access the service.

The Service object operates at Layer 4 of the OSI model which means that it only forward TCP and UDP connections and doesn't look inside of those connections.

[comment]: <> (TODO: Would be nice to link to OSI model here once you have that in your notes)

### Cloud Integration

If your cloud supports it, you can use the `LoadBalancer` type which builds on the `NodePort` concept. Essentially the cloud provider will create a load balancer and direct it at nodes in your cluster. This functionality is cloud provider specific, but is a way to get your application exposed/usable to the world.

### Endpoints

Some applications (and the system itself) want to be able to use services without a cluster IP. This can be done with the **Endpoints** object. For every Service object Kubernetes creates a buddy Endpoints object that contains the IP address of that service. You can view the Endpoints object with the command `kubectl describe endpoints alpaca-prod`. To use a service an application that is aware of Endpoints (likely an application written to work with Kubernetes) can talk to Kubernetes API directly to look up endpoints and call them. The Kubernetes API even has the capability to "watch" objects and be notified as soon as they change. This allows the client to react immediately as soon as the IPs associated with a service change. Most applications don't use this and just use stable IP addresses that don't change often, but the alternative is there.

### kube-proxy and cluster IPs

Cluster IPs are stable virtual IPs that load-balance traffic across all of the endpoints in a service. This is performed by a component running on every node in the cluster called `kube-proxy`. `kube-proxy` watches for new services in the cluster via the API server and then programs a set of `iptables` rules in the kernel of that host to rewrite the destinations of packets so they are directed at one of the endpoints for that service. If the set of endpoints for a service changes (due to Pods coming and going due to failed readiness checks) the set of `iptables` rules is rewritten.

The cluster IP itself is usually assigned by the API server as the service is created, however when creating the service the user can specify a specific cluster IP. Once set the cluster IP cannot be modified without deleting and recreating the Service object.

## Connecting with other environments

 When you are connecting Kubernetes to legacy resources outside of the cluster, you can use selector-less services to declare a Kubernetes service with a manually assigned IP address that is outside of the cluster. That way, Kubernetes service discovery via DNS works as expected, but the network traffic itself flows to an external resource.

 Connecting external resources to Kubernetes services is somewhat trickier. If your cloud provider supports it, the easiest thing to do is to create an “internal” load balancer that lives in your virtual private network and can deliver traffic from a fixed IP address into the cluster. You can then use traditional DNS to make this IP address available to the external resource. Another option is to run the full kube-proxy on an external resource and program that machine to use the DNS server in the Kubernetes cluster. Such a setup is significantly more difficult to get right and should really only be used in on-premise environments. There are also a variety of open source projects (for example, Hashicorp’s Consul) that can be used to manage connectivity between in-cluster and out-of-cluster resources.

## HTTP Load Balancing with Ingress

There are some challenges with using `NodePort` based Service object to expose your service outside of the cluster. For one since `NodePort` Service objects operate at Layer 4, you will need to have clients connecting on to a unique port per service. If you use a `LoadBalander` based Service object you will need to allocate an expensive cloud based load balancer outside your cluster for each service.

For Layer 7 (HTTP) applications we can do better. Outside of Kubernetes this problem is typically solved with a reverse proxy which decodes the incoming requests and sends them to the right upstream service. In Kubernetes this functionality is called **Ingress**. Ingress is a Kubernetes native way to implement the virtual hosting pattern described above. This virtual hosting pattern requires the administrator to manage the load balancer configuration file. In a dynamic environment as the set of virtual hosts expands, this can be very complex. Ingress simplifies this by standardizing that configuration, moving it to a standard Kubernetes object, and merging multiple Ingress objects into a single config for the load balancer.

The ingress controller is a software system exposed outside the cluster using a service type: `LoadBalancer`. It then proxies requests to "upstream" servers. The configuration of how it does this is the result of reading and monitoring Ingress objects.

## Ingress spec versus Ingress controllers

The implementation of Ingress is very different from other regular resource objects in Kubernetes. Ingress is split into a common resource specification and a controller implementation. There is not "standard" Ingress controller that is built into Kubernetes, and the user must install one of the many optional implementations. 

Users can create and modify Ingress objects just like every other object; but by default, there is no code running to actually oct on those objects. It is up to the users (or the distribution they are using) to install and manage an outside controller. In this way the, the controller is pluggable.

There are multiple reasons for this pluggable approach in Ingress. First there is no single HTTP load balancer that can universally be used. In addition to many software load balancers (both open source and proprietary), there are also load balancing capabilities provided by cloud providers such as ELB on AWS, and hardware based load balancers. The second reason is that Ingress was added to Kubernetes before the standard extensibility options. In the future Ingress may change to follow those new extensibility options.

## Configuring DNS

To make Ingress work well, you need to configure DNS entries to the external address of your load balancer. You can map multiple host names to a single external endpoint and the Ingress controller will direct incoming requests to the appropriate upstream service based on that hostname.

## Using hostname

The most common use of directing traffic based on properties of a request is using hostnames. The yaml file below defines an Ingress object that will route requests to `alpaca.example.com` to the alpaca service.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: host-ingress
spec:
  rules:
  - host: alpaca.example.com
  http: paths:
  - backend:
    serviceName: alpaca
    servicePort: 8080
```

## Default http backends

Certain Ingress controllers use a concept of a `default-http-backend`. This is used if a request comes in that does not have any http backend defined.  The default backend is uses as the default.

## Paths

You can use Ingress to direct traffic based on not just he hostname, but also the path in the HTTP request.  We can do this by specifying a path in the `paths` entry. In the example below, we are directing everything coming into `http://bandicoot.example.com` to the bandicoot service, but we send `http://bandicoot.example.com/a` to the alpaca service. This type of scenario can be used to host multiple services on different paths of a single domain.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: path-ingress
spec:
  rules:
  - host: bandicoot.example.com
  http:
    paths:
    - path: "/"
      backend:
        serviceName: bandicoot
        servicePort: 8080
    - path: "/a/"
      backend:
        serviceName: alpaca
        servicePort: 8080
```

When there are multiple paths on the same host listed in the Ingress system, the longest prefix matches. In the example above, traffic staring with /a/ is forwarded to the alpaca service, while all other traffic (starting with /) is directed to the bandicoot service.  As requests get proxied to the upstream service, the path remains unmodified. That means a request to `bandicoot.example.com/a/` shows up to the upstream server that is configured for that request hostname path.

## Advanced Ingress topics and gotchas

Many of Ingress' extended features are exposed via annotations on the Ingress object. Be careful, as these annotations can be hard to validate and are easy to get wrong. Many of these annotations apply to the entire Ingress object and so can be more general than you might like. To scope the annotations down you can always split a single Ingress object into multiple Ingress objects. The ingress controller should read them and merge them together.

## Running multiple Ingress Controllers

If you are running multiple Ingress controllers on a single cluster you can control which Ingress object is meant for which ingress controller using the `kubernetes.io/ingress.class` annotation. The value should be a string that specifies which Ingress controller should look at this object. The ingress controllers themselves, then, should be configured wth that same string and should only respect those Ingress objects with the correct annotation. If `kubernetes.io/ingress.class` annotation is missing, behavior is undefined. Its likely multiple controllers will fight to satisfy the Ingress and write the status field of the Ingress objects.

## Multiple Ingress Objects

If you specify multiple Ingress objects, the Ingress controllers should read them all and try to merge them into a coherent configuration. If duplicate or conflicting configurations are specified, the behavior is undefined.

## Ingress and namespaces

For security an Ingress object can only refer to an upstream service in the same namespace. This means that you can't use an Ingress object to point a subpath to a service in another namespace. On the other hand multiple multiple Ingress objects in the different namespaces can specify sub-paths for the same host. These Ingress objects are then merged together to come up with the final config for the Ingress controller. This cross-namespace behavior means that it is necessary that Ingress be coordinated globally across the cluster; otherwise an Ingress object in one namespace can cause issues for another namespace. Advanced users may try to force controls on this using admission controller, but there are no restrictions out of the box.

## Path rewriting

Some ingress controller implementations support, optionally, doing path rewriting. This can modify the path in the HTTP request as it gets proxied. This is typically specified with an annotation ont he Ingress controller (check the documentation for controller you are using). Be careful with path rewriting as if your web applications tries to link within itself using absolute paths the rewrite may cause issues.

## Serving TLS

To serve TLS via Ingress you will need to specify the certificate and key.  You can create the secret using `kubectl create secret tls <secret-name> --cert <certificate-pem-file> --key <private-key-pem-file>`. Once you have the certificate uploaded, you can reference it in an Ingress object. If multiple Ingress objects specify certificates for the same hostname, the behavior is undefined.

Check out [cert-mager](https://github.com/cert-manager/cert-manager) for setting up an API driven local certificate authority.

```yaml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
    - hosts:
      - alpaca.example.com
      secretName: tls-secret-name
rules:
- host: alpaca.example.com
  http:
    paths:
    - backend:
      serviceName: alpaca
      servicePort: 8080
```

## Alternate Ingress Implementations

Cloud providers have their own L7 based Ingress implementation that exposes the specific cloud based load balancer.  Instead of configuring the lod balancer to run in a Pod, these controllers take Ingress objects and use them to configure the cloud based load balancer via an API. This reduces load on the cluster, but you need to pay for the load balancer resources used.

The most popular generic Ingress controller is probably the open source [NGINX Ingress controller](https://github.com/kubernetes/ingress-nginx/). There is also a commercial version based on this project.   [Embassador](https://github.com/emissary-ingress/emissary) and [Gloo](https://github.com/solo-io/gloo) are options you should look at if you are looking to build an API gateway.  [Traefik](https://traefik.io) is a reverse proxy that can also function as an Ingress controller. It has a set of features and dashboards that are very developer-friendly.
