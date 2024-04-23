---
layout: page
title: "Date and Time"
permalink: /linux/date_time
---

## Date and Time

Two types of time are of interest to a process.

* Real time - Measured by Epoch the number of seconds since January 1, 1970 (UTC)
* Process time -the time spent executing code in kernel mode  (i.e. executing system calls and performing other kernel services on behalf of the process), and user CPU time the time spent executing code in user mode

The `time` command displays the real time, the system CPU time, and user CPU time taken to execute the process in a pipeline. For example `$time cat myfile.txt`
