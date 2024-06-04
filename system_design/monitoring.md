---
layout: page
title: "Monitoring"
permalink: /system_design/monitoring
---

## Statistics of Monitoring

* Alerting on a threshold is flawed.  You want to use statistics and compare to collected data in time series database.

## Server monitoring tools

* **Collectd** is a good tool for collecting data and writing to a time series database

## Application monitoring

* Look into **statsd** it's a common monitoring stack for applications.
* For microservices you want to use distributed tracing. Each request is tagged and you use the tag to trace the request as it hits each microservice.

The 4 golden signals according to Google SRE book

* latency
* traffic
* errors 
* saturation

Health checks are common and likely already being used if you are doing load balancing or Kubernetes with liveness probes or Consul.

For HTTP services you will want to track requests, successes and failures and time taken by each request.

Depending on your monitoring system tagging things such as which shard or which region or app version can be useful.

Latency and timings is a critical thing to monitor. You want to be as granular as you can here capturing as much as you can across your stack. Get request and response times from we servers, extend to load balancers and proxies, how much time are you spending decrypting SSL etc.

## Server Monitoring

### Memory

* Buffers and cached memory is essentially free.  The `-+ buffers` line in free command output show you the numbers minus cache and buffers.
* Alert on seeing OOM killer in logs. This is a good alert to set up. 

### Network

At a minimum you should collect

* octets in and out
* errors
* drops

### Disk

essentially you want to collect the `iostat` details which are...

* `await` is the average time in milliseconds for the issued command to be served by the disk.
* `%util` is saturation of the disk. This can be misleading if you are using raid. Want to keep this value below 100%
* `tps+` is transfers per second (IOPS) if IOPS over time drops you have a disk issue.

### Security monitoring

* sudo usage and ssh logins (from logs is one source of this)
* `auditd` can be used to track user actions (logins, file access, sudo calls). This can be a second layer of protection as a hacker may disable `rsynclog`.

#### NIDS (Network Intrusion Detection Systems)

NIDS use network taps to collect traffic. From these taps information is sent to a `SIEM` security information and event management system. Some open source examples are Snort and Bro.

## Services monitoring

### Checking if something did not happen

Below is a script that checks for something that did not happen such as a backup not running. There are more robust offerings out there.

It is generally a good idea to monitor cron job results

```bash
#!/bin/sh
    
    # Time in minutes
    TIME_LIMIT=$((60*60))
    
    # State file for updating last touch
    STATE_FILE=deadman.dat
    
    # Last access time of the state file (in epoch)
    last_touch=$(stat -c %Y $STATE_FILE)
    
    # Current time (in epoch)
    current_time=$(date +%s)
    
    # How much time is remaining before the switch fires
    timeleft=$((current_time - last_touch))
    
    if [ $timeleft -gt $TIME_LIMIT ]; then
      echo "Dead man's switch activated: job failed!"
fi
```

### NTP

You need to monitor for time drift. You can use exit code for `ntpstat` which returns error if time is not synced.

### Web servers

You want to track the following things

* requests per second
* http status codes
* process/response times based on logs
* 

### Databases

* Queries per second is an excellent indicator of how busy your applications is
* Slow queries from logs
* Number of concurrent threads/connections
* Replication delay and ensuring if replication is working
* Stats like table size, query timings, tuples read per query, returned etc. will help with debugging and predicting future performance regressions.

### Cache metrics

* You want to monitor for cache size, as well as hit/miss ratios and evict rate.
* miss ration should be as low as possible without risk of storing stale data.

### Queue Metrics

If you are using message queues to process jobs, you must monitor them.  Look for...

* Producer and consumer counts
* queue produce rates
* consumption rates
* lag and failure are critical numbers to track

Kafka and RabbitMQ publish a bunch of performance data that you will want to monitor.

### Business Metrics



## Frontend monitoring

Two approaches to front end monitoring

* Real monitoring where you have embedded JavaScript to send back metrics. Browsers expose metrics via navigation timing APIs (for example DOM complete - navigation start will give you the user perceived time to load)

* Synthetic (webpage.org for example)
