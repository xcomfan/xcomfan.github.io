---
layout: page
title: "Terminals"
permalink: /linux/terminals
---

## Pseudoterminals

A pseudoterminal is a pair of connected virtual devices, known as the master and slave. The device pair provides an IPC channel allowing data to be transferred in both direction between the two devices. The key point about a pseudoterminal is that the slave device provides an interface that behaves like a terminal, which makes it possible to connect a terminal-oriented program to the slave device and then use another program connected to the master device to drive the terminal-oriented program. Output written by the driver program undergoes the usual input processing performed by the terminal driver and passed as input to the terminal oriented program connected to the slave. In other words the driver program is performing the function usually done by a user sitting at a conventional terminal. Pseudoterminals are used most notable in the implementation of terminal windows provided under an X Window System login and in applications providing network login such as telnet and ssh.
