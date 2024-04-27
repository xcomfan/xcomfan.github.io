---
layout: page
title: "Trouble Shooting Reference"
permalink: /troubleshooting
---

[comment]: <> (TODO: Right now this section is one big info dump for me to review old notes.  Need to break it up into logical organization.)

## Linux Troubleshooting Commands

### strace

#### Attaching Multiple Processes to single strace output

`ps -auxw | grep sbin/httpd | grep -v grep | awk '{print"-p " $2}' | xargs strace`

##### Arguments

-f Trace child processes as they are created
-e Just show you the open

#### Problems people solve with strace

* Where is the config file?

* What other files does this program depend on?
  * Which .so file is it grabbing?
  * SSL root certificates
  * A games save files
  * which node_mdules are not being used?

* Why is this program hanging?
  * You can run strace -p PID and see what system call is currently running
  * May be polling with select()
  * may be wait() for a sub process to finish
  * making network requests to something that is not responding
  * doing a write() but it's blocked because the buffer is full
  * doing a read() on stdin and waiting for input
  * strace df -h you can find why df is hanging (may be a stuck mount that needs to be un-mounted)

* Why is this program stuck?
  * Variation of number 3.  If you see the program making system calls its not stuck.  Sometimes you just want to know if a program is stuck or not.
  * Why is this program slow?
  * strace -t will show the timestamp of each system call
    * You can look for big gaps and find the culprit

* Hidden permission errors
  * Sometimes when program is mysteriously not working it may be a permission issue that the program is failing to report.  You can figure this out by running the program with strace

* What command line arguments are being used
  * Sometimes a script will run another program.  You want to know what command line flags its passing.

* Why is this network connection failing?
  * You can find out which domain/IP address the network connection is being made to.  You can look at the DNS request or the connect() request.

* Why does this program succeed when run one way and fail when run in another way?
  * You can compare system calls being made to get a hint.

* How does this Linux kernel API work
  * See an example to supplement documentation.

* General reverse engineering.

### ps/top

`-p <pid1, pid2, …>`
`-u <uid>`
`-g <gid>`

### ss - socket statistics

| arg | purpose |
| --- | ------- |
| -t | Display the TCP socket family |
| -u | display the UDP sockets |
| -x | is for Unix domain sockets |
| -n | show port numbers instead of trying to resolve service names |
| -h | get help |

### grep

-n will print line numbers of on which line in the file there was a match

### curl

| arg | purpose | example |
| --- | ------- | ------- |
| -I | return only the HTTP headers of URL| |
| -o | output to a file | `curl -o myfile.html www.google.com` |
| -O | output to a file using its name as on server	| `curl -O www.google.com` |
| -H | modify your header | `curl -H "X-Header: value" https://www.keycdn.com` |
| -v | verbose | |
| -D | report download speed of | `curl -D - https://www.keycdn.com/ -o /dev/null` |
| -L | follow redirects	


#### Post call using curl

`curl -X POST http://www.yourwebsite.com/login/ -d 'username=yourusername&password=yourpassword'`

#### fetch multiple files

`curl -O URL 1 -O URL2`

## Important Tools for troubleshooting a problem per School of SRE

### For networking

* nc
* netcat
* traceroute
* mtr
* ping
* route
* tcpdump
* ss
* ip

### For DNS

* dig
* host
* nslookup

### For tracing system calls

* strace

### For parallel execution over ssh

* gnu parallel
* xargs + ssh

### HTTP checks

* curl
* wget

### For list of open files

* lsof

### For modifying attributes of the system kernel

* sysctl

### Performance analysis commands

* htop - like top, but a bit more interactive
* iotop
* vmstat
* iostat
* free
* sar
* mpstat
* perf

### Profiling Tools

* FlameGraph - Flame graphs are visualization of profiled software, allowing the most frequent code-paths to be identified quickly and accurately.

* Valgrind - Is a programming tool for memory debugging, memory leak detection and profiling.

* Gprof - GNU profiler tool uses a hybrid of instrumentation and sampling. Instrumentation is used to collect function call information, and sampling is used to gather runtime profiling information.

### Benchmarking

Benchmarking - is a process of measure the best performance of the service.  Like how much QPS service can handle, its latency when load is increasing, host resource utilization, loadavg etc.  The regression testing (i.e load testing) is a must before deploying the service to production.

#### Benchmarking tools

* Apache Benchmark Tool - Simulates high load on webapp and gather's data for analysis.

* Httperf - Sends request at a web server at a specified rate and gathers stats.  Increases till it finds the saturation point.

* Apache Jmeter - Open source measures web application performance

* Wrk: another tool to put load on your web server and give you latency, requests per second, transer per second, etc details.

* Locust: Easy to use scriptable and scalable performance testing tool.

* Python has a built in tool called tracemalloc that can help you review memory usage and spot memory leaks.
  * see for example here <https://linkedin.github.io/school-of-sre/level102/system_troubleshooting_and_performance/troubleshooting-example/>

## 10 System performance commands for troubleshooting a host

* `uptime` - you already knew everything they had to say about it.

* `dmesg | tail` - This views the last 10 system messages.  Look for any that may be performance related such as oom-killer.

* `vmstat` - show multiple statistics and at various intervals since boot
  * r - number of processes running on CPU and waiting for a turn.  Unlike uptime load averages this does not take runtime into consideration.  An r value greater than cpu count is saturation.
  * free - Free memory in kilobytes.
  * si, so: swap ins and swap outs.  You want these to be low.
  * us, sy, id, wa, st: These are breakdowns of CPU time, on average across all CPUs. User time system time (kernel) idle, wait I/O, and stolen time (by other guests or with Xen the guests own isolated driver domain) for VMs I guess.

* `mpstat -P ALL 1` - this command prints CPU time breakdowns per CPU, which can be used to check for an imbalance. A single host CPU can be evidence of a single threaded application.

* `pidstat 1` - like top for processes but rolling so that you can copy paste for investigation.  The CPU column is cumulative to a value of 1591% means that you are using nearly 16 cores.

* `iostat -xz 1` - This is a great tool for understanding block devices.
  * r/s, w/s, rkB/s, wKb/s: These are the delivered reads, writes, read Kbytes and write Kbytes per second to the device.  Use these for workload characterization.  
  * await: Average time for IO in milliseconds. Includes both time queues and time being serviced.
  * avgqu-sz: The average number of requests issues to the device.  Values greater than 1 can be evidence of saturation (although devices can typically operate on requests in parallel, especially virtual devices which front multiple back end disks)
  * %util - Device utilization.  This is really a busy percent, showing the time each second that the device was doing work.  Values greater than 60% typically lead to poor performance (which should be seen in await), Values close to 100% usually indicate saturation.
  * 100% IO may not necessarily be an issue.

* `free -m` 
  * buffers: For the buffer cache, used for block device I/O
  * cached: For the page cache, used by file systems.
  * Want to make sure that these numbers are not near 0 which can lead to higher disk I/O (confirm using `iostat`) and worse performance.  
  * The -/x buffers/cache provides less confusing values for used and free memory.  Linux uses free memory for the caches, but can reclaim it quickly if an application needs it, so cached memory should be included in the free memory column, which this line does.
  * ZFS has its own file system cache that is not reflected properly by free -m columns.  

* `sar -n DEV 1`
  * Use this tool to check network interface throughput: rxkB/s and txkB/s as a measure of workload and also to check if any limit has been reached.  
  * Also has a %ifutil for device utilization (max of both directions for full duplex)

* `sar -n TCP,ETCP 1`
  * This is a summarized view of some key TCP metrics.
  * active/s: number of locally-initiated TCP connections per second (e.g. via connect())
  * passive/s: number of remotely-initiated TCP connections per second (e.g. via accept())
  * retrans/s: Number of TCP retransmits per second.
  * The active passive counts are useful as a rough measure of server load.
  * retransmits are a sign of network or server issue. or may be due to a server being overloaded.

* `top`
