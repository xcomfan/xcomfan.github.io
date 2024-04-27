---
layout: page
title: "System Options"
permalink: /linux/system_options
---

## System Limits

You can use the `getconf` command to obtain the limits and options implemented by a particular UNIX implementation.

The general form of the command is `getconf variable-name | pathname |`

The variable name identifies the limit of interest for exmple...

```bash
$ getconf ARG_MAX
2097152
```
