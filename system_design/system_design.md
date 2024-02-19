---
layout: page
title: "System Design Reference"
permalink: /system_design
---

[comment]: <> (TODO: Need to break apart this section and I don't love system design as a top level catergory)

## Steps do designing your system

### Step 1: Requirements clarification

#### Functional Requirements

* The actual functionality: What can users do?  What can admins do? etc.
* Business rules
* Authentication

#### Non functional requirements

* High availability
* Consistency
* Number of users
* Latency for timelines etc

### Step 2: Estimation of important parts

* Storage
* Bandwidth Estimates

### Step 3: Data flow

* Rational DB or NoSQL DB

### Step 4: High level component design

### Step 5: Detailed Design

* Talk about tradeoffs

### Step 6: Identify and resolve bottlenecks

* Is there a single point of failure?
* Do you have enough replicas in case you have failures
* Do you have enough copies to prevent shutdown

## Back of envelope calculations

### Storage definitions

| bit | 1 or 0 |
| Byte | 8 bits |
| Kilobyte (KB) | 1024 bytes |
| Megabyte (MB) | 1024 Kilobytes |
| Gigabyte (GB) | 1024 Megabytes |
| Terabyte (TB) | 1024 Gigabytes |
| Petabyte (PB) | 1024 Terabytes |
| Exabyte (EB) | 1024 Petabytes |
| Zettabyte (ZB) | 1024 Exabytes |

### Storage estimates

| Single Char | 2 bytes |
| Unicode Char | 4 bytes |
| Long or Double | 8 bytes |
| Image or Photo | 200Kb |
| Good Photo | 2 BM |

### Operation timings

(one ns is one billionth of a second)

| Action | Time | Operations per second |
| ------ | ---- | --------------------- |
| L1 cache reference | 0.5 ns | 2 billion per second |
| L2 cache reference | 7 ns | 142 million per second |
| Mutex lock/unlock | 100 ns | 100 million per second |
| Main memory reference | 100 ns | 100 million per second |
| Compress 1K bytes | 10,000 ns | 100k per second |
| Send 2K bytes over 1 Gbps network | 20,000 ns | 50k per second |
| Read 1MB sequentially from memory | 250,000 ns | 4000 per second |
| Round trip within same datacenter | 500,000 ns | 2000 per second |
| Disk seek | 10,000,000 ns | 100 per second |
| Read 1MB sequentially from network | 10,000,00 ns | 100 per second |
| Read 1MB sequentially from disk | 30,000,000 ns | 33 per second |
| Send packet CA to Netherlands to CA | 150,000,0000,000 ns | 6.66 per second |

**Key takeaways**

* Compression can save a lot of network bandwidth
* Writes are 40 times more expensive than reads

## SQL vs no SQL

### Reasons to use SQL

* Structured data
* Strict schema
* Relational data
* Need for complex joins
* Transactions
* Clear patterns for scaling
* Lookups by index are very fast

### Reasons to use no SQL

* Semi structured data
* Dynamic or flexible schema
* Non relational data
* No need for complex joins
* Store many TB or PB of data
* Very data intensive workload
* Very high throughput for IOPs

### ACID

A: Atomic - All operations in a transaction succeed or every operation is rolled back.
C: Consistent - On completion of a transaction the database is structurally sound.
I: Isolated - Transaction do not contend with one another. Database makes sure that transactions appear to run sequentially.
D: Durable - Once the transaction is completed it will remain in system even if failure occurs.

### CAP Theorem

C: Consistency - All clients see the same data at the same time no matter which node they connect to.
A: Availability - Client gets a response if if one or more nodes are down.
P: Partition Tolerance - System continues to operate despite message loss or partial failure.

You basically need to pick two of these in a distributed system.


## Monitoring

### 4 Golder signals of monitoring

* Traffic - Gives an understanding of service demand
* Latency - Time taken to service an incoming request
* Error (rate) - measure of failed client requests
* Saturation - measure of resource utilization by a service

### Types of monitoring metrics

* Gauge
* Timer - time taken to complete a task
* Counter - number of occurrences of a particular event

## Fault tolerant metrics

| Abbreviation | Meaning |
| ------------ | ------- |
| MTTR | Mean time to repair |
| MTBB | Mean time between failures |
| MTTF | Mean time to failure |
| MTTD | Mean time to detect |
| MTTI | Mean time to investigate |
| MTRS | Mean time to restore service |
| MTBSI | Mean time between system incidents |

## Load Balancing

### Load Balance Algorithms

* Round Robin
* Sticky Round Robin - Keep sending to same host
* Weighted Round Robin
* IP/URL Hash
* Least Connections
* Least Response Time

## Caching

### Caching Strategies

