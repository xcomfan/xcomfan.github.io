---
layout: page
title: "Linux Standards"
permalink: /linux/standards
---

## POSIX

POSIX stands for portable operating system interface and it refers to a group of standards developed under IEEE.  The goal of the standard is to promote application portability at the source code level. POSIX documents an API for a set of services that should be made available to a program by a conforming operating system. POSIX xpecifies the API, but not the implementation.

## SUSvX

SUSvX stands for single UNIX Specification Version X and it is essentially POSIX 1003.1-2001 which replaces earlier POSIX standards.

## The Standard C Library; The GNU C Library

Determining the version of glibc on the system

From the shell, we can determine the version of glibc shared library files by running the .so file as though it was an executable. You will need to find where glibc is.

Another way is to run ldd against a linked program.

`ldd myprog | grep libc`
