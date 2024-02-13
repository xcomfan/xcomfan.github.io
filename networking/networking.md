---
layout: page
title: "Networking"
permalink: /networking
---

## TCP/IP

### Overview

A TCP entity accepts user data streams from local processes, breaks them up into pieces not exceeding 64KB (in practice 1460 data bytes in order to fit into a single Ethernet frame with the IP and TCP Headers), and sends each piece as a separate IP datagram. When datagrams containing TCP data arrive at a machine, they are given to the TCP entity, which reconstructs the original byte stream.

IP Layer gives no guarantee that datagrams will be delivered properly, nor any indication of how fast datagrams may be sent. It is up to TCP to send data-grams fast enough to make use of the capacity but not cause congestion, and to time out and retransmit any datagrams that are not delivered. Datagrams that do arrive may be in wrong order. It is up to TCP to reassemble them into messages in the proper sequence.

Two or more connections can terminate at the same socket. Connections are identified by socket identifiers at both ends.

Port numbers below 1024 are reserved for standard services and can only be started by root. They are called **well known ports**

TCP connections are full duplex (both nodes can send and receive data at the same time) and point to point (Each connection has only two endpoints).

A TCP connection is a byte stream, not a datagram stream. Message boundaries are not preserved end to end. For example; if the sending process does four 512 byte writes to a TCP stream, this data may be delivered to the receiving process as four 51 byte chunks, two 124 byte chunks or one 2058 byte chunk. As with Unix files, TCP software has no idea wha the bytes mean and no interest in finding out. A byte is just a byte.

The sending and receiving TCP entities exchange data in the form of segments. A **TCP segment** consists of a fixed 20 byte header (plus optional part) followed by zero or more data bytes. TCP software decides how big segments should be. It can accumulate data from multiple writes to a single segment or break u a write into multiple segments. Two limits to segment size are that TCP header must fit into the 65,515 byte IP payload, and each link has a Maximum MTU and each segment must fit into the MTU of the sender and receiver. TCP will use MTU discovery to find the minimum MTU size on a path and use that to avoid fragmentation (which leads to performance issues).

Basic protocol used by TCP is the sliding window protocol with dynamic window size. When a sender transmits a segment, it also starts a timer. When the segment arrived at the destination, the receiving TCP entity sends back a segment(with data if any exists, and otherwise without) bearing an acknowledgement number equal to the next sequence number it expects to receive and the remaining window size. If the sender's timer goes off before the acknowledgment is received, the sender transmits the segment again.