---
layout: page
title: "Kubernetes Nodes"
permalink: /kubernetes/nodes
---

## Nodes

Nodes are divided into **master** nodes and **worker** nodes. The master node will run containers such as the API server scheduler, etc. which manage the cluster. Kubernetes does not generally schedule work onto master nodes to ensure user workloads don't harm the overall operation of the cluster.
