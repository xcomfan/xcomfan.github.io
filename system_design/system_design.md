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


## L7 (applications load balancer has the following features)

* Examples of L7 load balancer are…
  * HAProxy
  * Envoy
  * Traefik

* Can forward request to healthy backends
* TLS termination
* HTTP routing
* header rewriting
* rate limiting of unauthenticated users
* Can keep connection to server open for lower resource use and latency
* requests can be retried in case of failure
* clients can use different IP protocols than servers
* Servers don't have to worry about MTU discover, TCP congestion control algorithms, avoiding the TIME-WAIT state and other low level details.

## How do you balance the traffic between load balancers?

* ExaBGP can be used to let load balancers announce their availability via BGP
  * It announces to the routers that load balancer is available
  * relies on L4 load balancing
  * If load balancer is removed you will have bad connections
  * Since routers choose their own routes there is not a lot of control here.

* L4 Load balancing
  * stateful - state synchronization among the members
  * stateless - consistent hashing

* DSR (Direct Server Return) When a server behind load balancer responds to the client directly bypassing the load balancer

* IPVS (IP virtual Server) this can give you a load balancing cluster (need to read up more on it)

* DNS Load balancing

## 7 Pipeline Design Patterns for Continuous Delivery

* Pipeline as Code
  * No GUI pipeline logic is managed just like application code
  * Execute on container to give everything its own environment
  * CI runner configuration is automated, identical, and hands free. CI runners can scale
  * Secrets are stored outside of the pipeline and their output is masked

* Externalize Logic into Reusable Libraries
  * Your CI tooling is software

* Separate Build and Deploy Pipelines
  * build once deploy many times
  * first build becomes an artifact that you can deploy many times

* Trigger the right pipeline
  * Pushing commit to pull request builds an Ephemeral Environment for testing
  * Merges to mainline go to a demo environment that is the latest code
  * Production pushes are signaled with tags
  * Fast Team Feedback
    * Every commit triggers a build with builds designed for speed and quick feedback
  * Stable Internal Releases
    * Only versioned packages produced by the build pipeline are deployed and these deployments are triggered by humans or automated events.
  * Buttoned up product releases
  * Deploy tagged releases to production and automate the paperwork but leave a paper trail.

## Notes on URL Resource Loading

* two types of loaders blocking and non blocking.  Blocking are ones that need to be loaded before a page can be displayed and non blocking are resources that can be loaded asynchronously in parallel with the main page (images and scripts)

* Loading process beings when the user requests a page by entering its URL in the browser.  The browser then sends a request to the sever, which sends back the HTML file, along with any other resources that the HTML file calls for. The browser then parses the HTML file and requests any additional resources that the hTML file calls for, such as images, stylesheets and scripts.

* Best practices for resource loading
  * Minimizing HTTP requests - this is more on how the website is built than on the resource loader.
  * Compression and minification
  * Caching and preloading - preloading is loading in background before they are actually needed.
  * Async and defer attributes - async attribute allows browser to load other stuff while loading this resource. Defer attribute specifies that a resource should be loaded only after the page has finished parsing, ensuring that the resource does not block the rendering of the page.
  * Lazy loading - load resources only when they are needed rather tha loading all resources at once.  
  
* Chromium design doc on resource loading
  * All network communication is done by the browser process.  This is done not only so that the browser process can control each renderer's access to the network, but also to maintain consistent session state across processes like cookies and cached data.  The browser as a whole should not open too many connections per host.

  * ResourceLoader implements the interface WebURLLoaderClient.  This is the callback interface used by the renderer to dispatch data and other events to Blink.  (Blink is the render engine that renders web pages)

  * Requests are sent to a global ResourceDispatcherHost via IPC communication.

  * Cookies are handled in browser process which handles all network requests becuase cookies need to be the same across all tabs.  When page requests a cookie an async message is sent from the renderer to the browser requesting the cookie.  While the browser is processing the cookie the thread that Blink works on is suspended.  When the renderer's I/O thread receives the response from the browser, it un-suspends the thread and passes the result back to the JavaScript engine.

  * You can scan the html in two passes one for the blocking and one for the non blocking and fire off the blocking first then non blocking but only return when non blocking is done.