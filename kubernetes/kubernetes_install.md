---
layout: page
title: "Kubernetes Installation"
permalink: /kubernetes/installation
---

## KCA book chapter 2 process

Since the script in the book does not work here is the version that I created based on docs.

```bash
apt-get update
apt-get install -y docker.io
service docker restart
apt-get install -y apt-transport-https ca-certificates curl gnupg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # allow unprivileged APT programs to read this keyring
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' > /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list   # helps tools such as command-not-found to work correctly
apt-get update
apt-get install -y kubeadm kubectl kubelet
```

## Content of the kube.sh script once fixed

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml

## Chapter 2 Lab notes

### Lab 2

### Init cluster

`kubeadm init --pod-network-cidr=192.168.0.0/16`

This gives you the command to run on the second host.

### Kubectl config to point to your cluster

```bash
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

### Add ingress controller and mysterious calico kube controller

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml
```

### Lab 3

use the pv.yaml file in your repo and run the command `kubectl create -f pv.yaml`


Example of sorting the volumes by how much storage capacity they have. `kubectl get pv --sort=.spec.capacity.storage`


### Lab 4 - Performing a Kubernetes upgrade with kubeadm

#### Control Plan Upgrade Steps

Because the book I am using sucks, I need to do some hacking with versions to be able to rproduce the lab so all the commands below are just notes, I did not verify them.

* Upgrade kubeadm on the control plane node. 
* Drain the control plane node `kubectl drain ip-172-31-21-95 --ignore-daemonsets`
* Plan the upgrade (kubeadm upgrade plan)
* Apply the upgrade (kubeadm upgrade apply) `sudo apt-get update && sudo apt-get install -y --allow-change-held-packages kubeadm=1.27.2-00 && kubeadm upgrade apply v1.27.2`
* Upgrade kubelet and kubectl on the control plan node `sudo apt-get install -y --allow-change-held-packages kubectl=1.27.2-00 kublet=1.27.2-00 && systemctl daemon-reload && systemctl restart kubelet`
* Uncordon the control plane node `kubectl uncordor ip-172-31-21-95`

#### Worker node upgrade steps

* Drain the node
* Upgrade kubeadm `sudo apt-get install -y --allow-change-held-packages kubeadm=1.27.2-00`
* Upgrade the kubelet configuration (kubeadm upgrade node) `sudo kubeadm upgrade node`
* Upgrade kubelet and kubectl `sudo apt-get install -y --allow-change-held-packages kubectl=1.27.2-00 kubelet=1.27.2-00 && systemctl daemon reload && systemctl restart kubelet`
* Uncordon the node `kubectl uncordon HOSTNAME`


#### Lab 5 - Working with namespaces

Each Kubernetes resource must be in only one namespace and namespaces cannot be nested inside one another.

You can define resource quotas and limits within each namespace to ensure that one tenant cannot monopolize cluster resources. This involves setting CPU, memory and storage limits for each tenant's namespace.

To create a namespace named `dev` use the command `kubectl create namespace dev`

## KCA book chapter 3 notes

For HA in Kubernetes you need to have an multi master cluster.

Kubernetes datastore is, by default, Dqlite, a high availability SQLite.

For HA to work you need to have multiple instances of Kubernetes API running and that means that they need to be behind a load balancer.

**Stacked etcd** If you have an HA setup with control plan running on multiple nodes, then you have etcd running on those nodes and that is stacked etcd. I guess the other option is to have etcd running on other nodes. There is a risk here of loosing both etcd and control plan if a node goes down, but that can be mitigated by adding more hosts.  

**External etcd** This is just etcd running on nodes other than your control plane nodes.

### K8s management tools intro

#### kubeadm

`kubeadm` utility provides `kubeadm init` and `kubeadm join` commands as best-practice quick path ways of establishing Kubernetes clusters.

#### minikube

Lets you set up a cluster using just a single server or machine for practice.

#### helm

Helm provides templating and package management for Kubernetes objects. You can turn your Kubernetes objects or the applications running in your cluster into templates, referred to you by helm as charts to allow you to easily manage complex configurations of multiple Kubernetes objects. You can also download and use shared templates that others have already created.

helm is a package manager that allows you to search, share and use Kubernetes-specific applications. It simplifies the installation and maintenance of Kubernetes apps and is the Kubernetes equivalent of apt or yum.

helm uses a chart packaging format. A chart is a series of files that explain a group of Kubernetes resources. A single chart might be used to deploy anything basic, such as memcaced pod, or something sophisticated, sch as a whole we b app stack with HTTP servers, databases and caches.

#### kompose

Kompose is a tool that can convert docker compose files into kubernetes objects to help you move from local development to running in Kubernetes.

#### kustomize

Kustomize is like helm, but it has a concept of layering where you have a base version and layering yaml artifacts (referred to as patching). This is a strong approach as most firms construct their project from a set of internally developed and off the shelf apps. You can leverage kustomize to absorb any base file modifications for your underlying components while preserving use-case-specific customization overrides. 

### Safely Draining a K8s Node

#### Draining

Draining a node will cause the containers running causes containers running on the node to be gracefully terminated and moved to another nodes to prevent service interruption so that you can do maintenance on a node. When a node is being drained Kubernetes will also prevent fresh pods from coming in. If there are deamon set managed pods, the drain does not proceed unless the `--ignore-demonsets` option is used. Unless you use `--force`, the drain does not destroy any pods that do not mirror pods or are handled by a replication controller or are handled by a replicatiosn controller, replica set, DaemonSets, stateful set, or job. If the management resource of one or more pods is absent, `--force` enabled deletion to occur.

Format of the drain command is `kubectl drain <node name>`

When you are ready for the node to operate again use `kubectl uncordon` to make the node schedulable again.

`kubectl drain` by default ignores system pods on the node that cannot be destroyed.

### Upgrading K8s with kubeadm

#### Control plane upgrade steps

Following are the steps to upgrade the control plan node:

* Upgrade kubeadm on the control plane node
* Drain the control plane node
* Plan the upgrade (kubeadm upgrade plan)
* Apply the upgrade (kubeadm upgrade apply)
* Uncordon the control plane node

#### Recovering from a Failure State

If the kubeadm upgrade fails and does not roll back for example if you lost power or network during execution you can perform it again, the command is idempotent. You can also try `kubeadm upgrade apply --force` to recover from a problematic condition without altering the version of your cluster.

During the upgrading process, kubeadm creates the following backup directories in `/etc/kubernetes/tmp`

* `kubeadm-backup-etcd-<date>-<time>` - backup of this control plane nodes local etcd member data. If upgrade fails contents of this folder can be manually restored in /var/lib/etcd.
* `kubeadm-backup-manifests-<date>-<time>` - copieds this control plane node's static Pod manifest files. If an upgrade fails and the automated reollback also fails, the contents of this folder can be manually restored in `/etc/kubernetes/manifests`.

