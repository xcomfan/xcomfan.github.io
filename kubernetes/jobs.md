---
layout: page
title: "Kubernetes Jobs"
permalink: /kubernetes/jobs
---

## What are Kubernetes Jobs

The Job construct in kubernetes lets you run one time short-lives task. A job creates Pods that run until successful termination (i.e. exit with 0).

## The Job object

The Job object is responsible for creating and managing Pods defined in a template in the job specification. These Pods generally run until successful completion. The Job object coordinates running a number of Pods in parallel. If the Pod fails before a successful termination, the job controller will create a new Pod based on the pod template in the job specification.

Given that Pods have to be scheduled, theres is a chance that your job will not execute if the required resources are not found by the scheduler. Also, due to the nature of distributed systems there is a small chance, during certain failure scenarios, that duplicate Pods will be created for a specific task.

## Job patterns

Jobs are designed to manage batch-like workloads where work items are processed by one or more Pods. By default, each job runs a single Pod once until successful termination. This job pattern is defined by two primary attributes of a job, namely the number of job completions and the number of Pods to run in parallel. In the case of run once until completion for example, the `completions` and `parallelism` parameters are set to 1.

| Job pattern type | Use case | Behavior | Completions | Parallelism |
| ---------------- | -------- | -------- | ----------- | ----------- |
| One shot | Database migrations | A single Pod running once until successful completion | 1 | 1 |
| Parallel fixed completions | Multiple pods processing a set of work in parallel | Once or more Pods running one or more times until reaching a fixed completion count | 1+ | 1+ |
| Work queue | Multiple Pods processing from a centralized work queue | One or more Pods running once until successful termination | 1 | 2+ |

### One shot

One shot jobs provide a way to run a single Pod once until successful termination. A Pod must be created and submitted to the Kubernetes API. This is done using a Pod template defined in the job configuration. Once a job is up and running, the Pod backing the job must be monitored for successful termination. A job can fail for any number of reasons, including an application error, an uncaught exception during runtime, or a node failure before the jbo has a chance to complete. In all cases, the job controller is responsible for recreating the Pod until a successful termination occurs.

There are multiple ways to create a one-shot job in Kubernetes. The easiest is to use the `kubectl` command line tools for example...

`kubectl run -i oneshot --image=gcr.io/kuar-demo/kuard-amd64:blue --restart=OnFailure -- --keygen-enable --keygen-exit-on-complete --keygen-num-to-gen 10`

Some things to note about the above example

* The `-i` option indicates that this is an interactive command. `kubectl` will wait until the job is running and then show the log output from the first (and in this example only) Pod in the job.

* `--restart=OnFailure` is the option that tells `kubectl` to create a Job object. Restart policy can be set to Never if you do not wish for kubernetes to keep trying to restart the Pod if it fails. The behavior with Never will be to keep crating new Pods which can create a lot of Junk in your cluster so the `OnFailure` restart option is preferred. Kubelet has mechanisms to do a crash loop backoff so as to not keep eating resources on the cluster.

* All of the options after `--` are command line arguments to the container image. These instruct our test server (kuard) to generate 10 4096 bit SSH keys and then exit.

* `kubectl` often misses the first couple of lines of the output with the `-i` option.

After the job has completed, the Job object and related Pod are still around. This is so that you can inspect the log output. Note that this job won't show up in `kubectl get jobs` unless you pass the `-a` flag. Without this flag `kubectl` hides completed jobs. To delete the job use the command `kubectl delete jobs oneshot`.

You can also use a manifest file to create a job. Below is an example. You would then submit the job with the command `kubectl apply -f job-oneshot.yaml`. You can view the results of the job by looking at the logs of the Pod that was created using `kubectl describe jobs oneshot` to find the Pod and `kubectl logs oneshot-4kfdt` to see the logs from Pod.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: oneshot
spec:
  template:
    spec:
      containers:
      - name: kuard
        image: gcr.io/kuar-demo/kuard-amd64:blue
        imagePullPolicy: Always
    args:
    - "--keygen-enable"
    - "--keygen-exit-on-complete"
    - "--keygen-num-to-gen=10"
    restartPolicy: OnFailure
```

One thing to consider is a job may get stuck and not make progress. To handle this you want to use a liveness probe to determine if the job should be restarted.

### Parallelism

If for example you wnt to generate 100 keys by having 10 runs of kuard with each generating 10 keys while limiting to just running 5 pods at a time this will translate to `completions` of 10 and `parallelism` of 5. Below is an example of such a job manifest. You can start this job with the command `kubectl apply -f job-parallel.yaml`

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel
  labels:
    chapter: jobs
spec:
  parallelism: 5
  completions: 10
  template:
    metadata:
      labels:
        chapter: jobs
    spec:
      containers:
      - name: kuard
        image: gcr.io/kuar-demo/kuard-amd64:blue
        imagePullPolicy: Always
        args:
        - "--keygen-enable"
        - "--keygen-exit-on-complete"
        - "--keygen-num-to-gen=10"
      restartPolicy: OnFailure
```

### Work Queues

The manifest example below is telling the job to start up five Pods in parallel. As the `completions` parameter is unset, we put the job into a worker pool mode. Once the first Pod exits with a zero exit code, the job will start winding down and will not start any new Pods. This means that none of the workers should exit until the work is done and they are all in the process of finishing up. Note that in the book a work to be done queue was set up for the example workers.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  labels:
    app: message-queue
    component: consumer
    chapter: jobs
  name: consumers
spec:
  parallelism: 5
  template:
    metadata:
      labels:
        app: message-queue
        component: consumer
        chapter: jobs
    spec:
      containers:
      - name: worker
        image: "gcr.io/kuar-demo/kuard-amd64:blue"
        imagePullPolicy: Always
        args:
        - "--keygen-enable"
        - "--keygen-exit-on-complete"
        - "--keygen-memq-server=http://queue:8080/memq/server"
        - "--keygen-memq-queue=keygen"
      restartPolicy: OnFailure
```

For the example manifest above we can clean up using labels with the command `kubectl delete rs,svc,job -l chapter=jobs`

### CronJobs

Below is an example of a CronJob definition in Kubernetes. Note the `sepc.schedule field`, which contains the interval for the CronJob in standard cron format. As with other definitions you can schedule it with the command `kubectl create -f cron-job.yaml` and use `kubectl describe <cron-job>` to get the details.

```yaml
apiVersion: batch/v1beta1
    kind: CronJob
    metadata:
      name: example-cron
    spec:
      # Run every fifth hour
      schedule: "0 */5 * * *"
      jobTemplate:
        spec:
          template:
            spec:
              containers:
              - name: batch-job
                image: my-batch-image
              restartPolicy: OnFailure
```
