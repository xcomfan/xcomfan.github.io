---
layout: page
title: "Installing and configuring Terraform"
permalink: /install_and_configure
---

## AWS Configuration

You need to have the below environment variable set for Terraform to be able to access your AWS account.

```bash
$ export AWS_ACCESS_KEY_ID=(your access key id)
$ export AWS_SECRET_ACCESS_KEY=(your secret access key)
```

## Configuring Kubectl To Connect To EKS Cluster

```bash
aws sts get-caller-identity
aws eks update-kubeconfig --region <region_from_terraform_output> --name <name_from_terraform_output>
kubectl get svc # To verify
```

Once connected you may want to create and set namespace.

`kubectl create namespace <namespace_name>`

And to configure kubectl to use the context you need to create the context and make it the default.

With AWS I am doing this by modifying the one they create. So command looks like

`kubectl config get-contexts` to get the details fo current context than command below with the name to set the default namespace.

```bash
kubectl config set-context <context_name> --namespace <name_of_namespace>
kubectl config use-context <context_name>
```
