---
layout: page
title: "Kubernetes Deployments"
permalink: /kubernetes/deployments
---

[comment]: <> (TODO: Need to review and organize this section)

## What are Deployments

The Deployment object exists to manage the release of new versions. Deployments enable you to easily move from one version of your code to the next.

## Deployment Spec

Like with all Kubernetes objects, a deployment can be represented as a declarative YAML object that provides the details about what you want to run. Below is a manifest for a single instance of the `kuard` application.

[comment]: <> (TODO: Have your own version not kuard here once you run though exercises)

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: kuard
spec:
  selector:
    matchLabels:
      run: kuard
    replicas: 1
    template:
      metadata:
        labels:
          run: kuard
      spec:
        containers:
        - name: kuard
          image: gcr.io/kuar-demo/kuard-amd64:blue
```

To run the deployment specified by the above script you would run the command `kubectl create -f kuard-deployment.yaml`

## Deployment internals

Similar to how ReplicaSets manage Pods; deployments manage ReplicaSets. As with all relationships in Kubernetes the deployment to ReplicaSets relationship is defined by labels and a label selector. You can see the label selector by looking at the Deployment object with the below command.

`kubectl get deployments kuard -o jsonpath --template {.spec.selector.matchLabels}`

You can imperatively scale a deployment using the command `kubectl scale deployments kuard --replicas=2`.  ***Note:*** If you want to take any actions on the ReplicaSets managed by a deployment you need to delete the deployment. Otherwise the self healing nature of Kubernetes will keep taking actions to align the ReplicaSets with the stored specification for the deployment.

[comment]: <> (TODO: Need to play around with this concept as I don't belive that satement is 100% ture. You can change parts of a deployment and I don't think it will auto heal it, but need to play around)

## Creating deployments

As a starting point you can generate the YAML manifest for an imperatively declared deployment to create a declarative one.

To get the deployment as a YAML file use the following command...

`kubectl get deployments kuard --export -o yaml > kuard-deployment.yaml` followed by `kubectl replace -f kuard-deployment.yaml --save-config`

The reason you need to run the second command (with --save-config) is that it adds an annotation so that when applying changes in the future, kubectl will know what the last applied configuration was for smarter merging of conflicts. If you always use `kubectl apply`, this step is only required after the fist time you create a deployment using `kubectl create -f`.

[comment]: <> (TODO: Need to work though the example above as I don't quite follow the explanation of the second command)

The deployment spec has a very similar structure to the ReplicaSet sepc. There is a Pod template which contains a number of containers that are created for each replica managed by the deployment. In addition to the Pod specification, there is also a `strategy` object.

```yaml
...
strategy:
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
  type: RollingUpdate
...
```

The `strategy` object dictates the different ways in which a rollout of the new software can proceed. There are two strategies supported: `Recreate` and `RollingUpdate`

## Managing deployments

To get information about a deployment use the command similar to `kubectl describe deployments kuard`

From the output of the above command, two of the most important pieces of information are `OldReplicaSets` and `NewReplicaSets`. These fields point to the ReplicaSet object this deployment is currently managing. if a deployment is in the middle of a rollout, both fields will be set to a value. If a rollout is complete, `OldReplicaSets` will be set to `<none>`.

You can also use the command `kubectl rollout history` to obtain the history of rollouts associated with a particular deployment. If you have a current deployment in progress you can use `kubectl rollout status` to obtain the current status of a rollout.

### Scaling a deployment

To scale up a deployment, you would edit your YAML file to increase the number of replicas. Once you have saved and committed the change, you can use the `kubectl apply` command to put it into effect.

### Updating a container image

Updating a container is also an update to a YAML file and a `kubectl apply` command call. It is a good idea to add an annotation to the deployment to record some information about the update.  Make sure to add this annotation to the template and not the deployment in the YAML file as `kubectl apply` command uses the field in the Deployment object. Also do not update the `change-cause` annotation when scaling as a change to that annotation will trigger a new rollout.

Once you make the change and call `kubectl apply` you can monitor the rollout via the `kubectl rollout` command.

If you are in the middle of a rollout and you want to temporarily pause the rollout (if you are seeing issues and want to investigate for example) you can use the command similar to `kubectl rollout pause deployments kuard`. To resume the rollout use `kubectl rollout resume deployments kuard`.

### Rollout history

You can see the deployment history by running the command `kubectl rollout histoyr deployment kuard` If you are interested in more details about a particular revision, you can add the `--revision` flag as in `kubectl rollout history deployment kuard --revision=2`

If you need to undo a rollout to the prior version you can use the command `kubectl rollout undo deployment kuard`.The undo command works regardless of the state of the rollout. You can undo both partially completed and fully completed rollouts. ***Note***: This is the imperative approach. The preferred declarative way is to update the YAML file to the version you are rolling back to and run that deployment.

By default the revision history of a deployment is kept attached to the Deployment object. If you have deployment that you plan to keep for a long time and you make frequent updates you should limit the revision history. You can do this by setting the `revisionHistoryLimit` property in the deployment specification.

## Deployment Strategies

As mentioned prior, Kubernetes supports two different rollout strategies, `Recreate` and `RollingUpdate`

### Recreate Strategy

The recreate strategy simply updates the ReplicaSet it manages to us the new image and terminates all of the Pods associated with the deployment. The ReplicaSet notices that it no longer has any replicas, and re-creates all Pods using the new image. Once the pods are re-created, they are running the new version. This strategy will almost certainly result in some downtime. Its typically uses for test deployments.

### RollingUpdate strategy

RollingUpdate can be used to roll out a new version of your service while it is still receiving user traffic, without any downtime. It works by updating a few Pods at a time until all of the Pods are running the new version of your software.

This means that for a period of time, both the new and the old version of your service will be receiving and serving traffic. The software you are deploying needs to support this state.

#### Configuring a rolling update

There are two parameters you can use to tune the rolling update behavior; `maxUnavailable` and `maxSurge`.

The `maxUnavailable` parameter sets the maximum number of Pods that can be unavailable during a rolling update. It can either be set to an absolute number (3 for example meaning maximum of 3 Pods can be unavailable) or as a percentage (20% meaning a maximum of 20% of the desired number of replicas can be unavailable). Generally you would use percentage, but setting hard value of 1 lets you deploy one Pod at a time.

Using reduced capacity is one option of doing a deployment, but it may not always be a valid option (if you do not have dips in traffic for example). The other option is to create extra instances and take out the old ones which is done with the `maxSurge` parameter. The `maxSurge` parameter controls how many extra resources can be created to achieve a rollout. For example you can set `maxUnavailable` to 0 and `maxSurge` to 20%. The rolling update will scale up by 20% first then start removing the old instances.

Setting `maxSurge` to 100% is equivalent to a blue/green deployment. The deployment controller first scales the new version to 100% of the old version. Once the new version is healthy, it immediately scales the old version down to 0%

#### Slowing rollouts to ensure service health

The deployment controller examines the Pod's status with readiness checks. If you want to use deployments to reliably roll out your service you must implement health checks for the containers in your Pod. Without these checks deployment controller is running blind.

Just because a pod comes up healthy does not mean it does not have any issues. Memory leaks or low frequency bugs can take some time to be noticed. In most real world scenarios you want to wait some time before moving on to the next Pod. For deployments this time to wait can be defined by the `minReadySeconds` parameter. Setting `minReadySeconds` to 60 means a deployment has to wait for 60 seconds after a Pod comes up healthy until it moves on to the next one.

You also want to set a timeout for the rollout should you have a scenario where Pods are just not coming up healthy. If you are using automated system to deploy a timeout is a good place to fire off a ticket and roll back the deployment. To set a timeout use the `progressDeadlineSeconds` parameter. Its important to note that his setting is for deployment progress not the overall length of a deployment. Every time the deployment creates or deletes a Pod the clock is reset for the timeout.

## Deleting a Deployment

To imperatively delete a deployment ue the command similar to `kubectl delete deployment kuard`. To declaratively delete a deployment `kubectl delete -f kuard-deployment.yaml`.

Deleting a deployment deletes the entire service including the ReplicaSets and Pods. If you want to leave those intact you can use the `--cascade=false` option to only delete the Deployment object.

## Monitoring a deployment

The status of a deployment can be obtained from the `status.conditions.array` where there will be a `Condition` whose `Type` is `Progressing` and whose `Status` is `False`. A deployment in such a state has failed and will not progress further.  How long you should wait for this state is controlled by the `spec.progressDeadlineSeconds` property.
