---
layout: page
title: "Date and Time"
permalink: /linux/date_time
---

## Date and Time

Two types of time are of interest to a process.  **Real time** and **Process time**

* Real time - Measured by Epoch the number of seconds since January 1, 1970 (UTC).  This internal time is computed this way regardless of location.

### Process time

Process time is the amount of CPU time used by a resource since it was created.  Sometimes process time is referred to as the total CPU time consumed by the process.  The kernel separates CPU time into the following components.

* User CPU time - is the amount of time spent executing in user mode. Sometimes referred to as virtual time this is the time that appears to the program that it has access to the CPU

* System CPU time - is amount of time spent executing in kernel mode.  This is the time that the kernel spends executing systems calls or performing other tasks on behalf of the program.  

When we run a program from the shell we can use the `time` command to obtain both process time values as well as the real time required to run the program.

```bash
$ time ./myprog
real    0m0.007s
user    0m0.001s
sys     0m0.005s
```

## Timezones

Timezone information is kept in definition files located at `/usr/share/zoneinfo`. Each file in this directory contains information about the timezone regime in a particular country or region.

To specify a timezone when running a program, we set the TZ environment variable to a string consisting of a colon (:) followed by one of the timezone names defined in `/usr/share/zoneinfo`.

## Locales

Locale files have information such as what the date format should be for a region as well as other language and cultural conventions.

Locale definitions are kept in `/usr/lib/locale` or `/usr/share/locale` (depending on distribution)

Each subdirectory under this directory contains information about a particular locale.  These directories are named using the following convention:

`language[_territory[.codeset]]@modifier` for example `de_DE.tf-8@euro`

* language is a two letter ISO language code
* territory is a two letter country code
* codeset designates a character encoding set
* modifier provides a means of distinguishing multiple locale directories whose language codeset and territory are the same.
* If the name for locale is not an exact match the c library will strip components in the following order.
  1. Codeset 
  2. Normalized codeset
  3. Territory
  4. modifier

a normalized codeset is a version of the codeset name in with all non alphanumeric character are removed.

## The Software Clock (Jiffies)

The accuracy of various time related system calls is limited to the resolution of the system software clock, which measures time in units called `jiffies`

The size of a `jiffy` is defined by the constant HZ within the kernel source code. This is the unit in which the kernel allocates the CPU to processes under the round-robin time sharing scheduling algorithm.

The advantages of a higher software clock rate are that timers can operate with greater accuracy and time measurements, can be made with greater precision. However it isn't desirable to set the clock rate to arbitrarily high values because each clock interrupt consumes a small amount of CPU time, which is time that the CPU can't spend executing processes. This has become a configurable kernel option.
