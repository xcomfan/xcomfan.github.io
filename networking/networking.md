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

### TCP Segment Structure

A TCP segment consists of data bytes and a header that is added to the data by TCP. The header of a TCP segment can range from 20 to 60 bytes. 40 bytes are for options. If there are no options, a header is 20 bytes.

| Content | Size | Purpose |
| ------- | ---- | ------- |
| Source port address | 16 bits | Port address of application sending the data segment |
| Destination port address | 16 bits | Port address of the application in the host receiving data segment |
| Sequence number | 32 bits | Byte number of the fist byte that is sent in that particular segment. Used to assemble segments received out of order. |
| Acknowledgement number | 32 bits | Byte number receiver expect to receive next. In other words its the next in order byte expected. Its an acknowledgement for previous bytes being received successfully. |
| Header Length (HLEN) | 4 bits | Indicates length of TCP header by a number of 4 byte words. For example if header is 20 bytes (min length of TCP header) then this field holds 5 (because 5 * 4 is 20) and the maximum 60 bytes will be 15 (15 * 4 is 60). Value of this field is always between 5 and 15. Tells how many 32 bit words are contained in the TCP header. Needed because the option field is variable in length so the header field is as well. |
| Control Flags | 6 bits | 6 1 bit control bits that control connection establishment. More details below |
| Windows Size | 16 bits | Tells the window size of the sending TCP in bytes. 0 size is legal and says that the bytes upt o and including Acknowledgment number -1 have been received, but that the receiver has not had a chance to consume the data and would like no more data for the moment. Receiver can send another message later with a non zero window sze to continue the communication. |
| Checksum | 16 bits | Holds the checksum for error control |
| Urgent Pointer | 16 bits | Valid only if URG control flag set. Used to point to data that is urgently required that needs to reach the receiving process at the earliest. The value of this field is added to the sequence number to get the byte number of the last urgent byte |
| Options | 0 or more 32 bit words | Options allow to add extra facilities not covered by regular header. Options are variable in length and use 0 padding to fill multiples of 32 bits. One options example is maximum segment size used during connection establishment. Without this option you default to a 536 byte payload. Timestamp option is another example which is used to solve the issue of sequence number wrapping around. SACK (Selective Acknowledgment Option ) lets a receiver tell the sender the ranges of sequence numbers it has received. This is a supplement to acknowledgement numbers and is used after a packet has been lost, but subsequent or duplicate data has arrived |

#### Control Flag details

| CWR | Congestion Window Reduced: Set to signal that congestion windows reduced from TCP sender to TCP receiver so that it knows the sender has slowed down and can stop sending ECN echo.|
| ECE | When congestion notification is triggered ECE is to to ECN-Echo to TCP sender to tell it to slow down|
| URG | Urgent pointer is valid |
| ACK | Acknowledgement number is valid (for cumulative acknowledgement) |
| PSH | Request for push. Kindly ask receiver to send to app directly and not wait for buffer to fill up|
| RST | Reset connection or refuse to open connection |
| SYN | Synchronize sequence numbers. Connection request has SYN = 1 and Ack = 0 to indicate that piggyback acknowledgement is not in use. The connection reply does have an acknowledgement so it has SYN = 1 and ACK = 1|
| FIN | Terminate connection. Means sender has no more data to transmit |

### TCP Connection Establishment (3 way handshake)

To establish a connection, one side (the server) passively waits for an incoming connection (LISTEN and ACCEPT) primitives.

The other side (the client) executes the CONNECT primitive specifying the IP address and port to which it wants to connect, the maximum TCP segment size it is willing to accept and optionally some user data (e.g password). The CONNECT primitive sends a TCP segment with SYN bit on and the ACK bit off and waits for a response from the other end.

When the segment arrives at the destination, the entity there checks to see if there is a process that has done a LISTEN on the port given in the destination. If not it sends a reply with the RST bit on to reject the connection. If some process is listening to the port, the process is given the incoming TCP segment. It can either accept or reject the connection. If it accepts the acknowledgement segment is sent back.

1. Host 1 sends SYN (SEQ = x)
2. Host 2 sends SYN (SEQ = y, ACK = x + 1)
3. Host 1 sends (SEQ = x =1, ACK = y + 1)

### TCP Connection Release

Even though TCP connections are full duplex, to understand how connections are released its easier to think of the connection as a pair of simplex connections. Each simplex connection is released independently of its sibling. To release a connection either party can send a TCP segment with the FIN bit set, which means it has no more data to transmit. When the FIN is acknowledged that direction is shut down for new data. If response to FIN is not received within 2 packet lifetimes, the sender of the FIN releases the connection. The other side will eventually notice that nobody seems to be listening to it anymore and will time out as well.

### States of TCP Connection

| State | Description |
| ----- | ----------- |
| CLOSED | No connection is active or pending |
| LISTEN | The server is waiting for an incoming call |
| SYN RCVD | A connection request has arrived; wait for ACK |
| SYN SENT | The application has started to open a connection |
| ESTABLISHED | The normal data transfer state |
| FIN WAIT 1 | The application has said it is finished |
| FIN WAIT 2 | The other side has agreed to release |
| TIME WAIT | Wait for all packets to die off |
| CLOSING | Both sides have tried to close simultaneously |
| CLOSE WAIT | The other side has initiated a release |
| LAST ACK | Wait for all packets to die off |

### TCP Sliding Window

### TCP Congestion Protocol

## TLS

### What happens during a TLS handshake

During a TLS handshake the client and server will do the following.

* Specify which version of TLS they will use (1.0, 1.2, 1.3 etc)

* Decide on which cipher suites they will use.

* Authenticate the identity of the server via the server's public key and the SSL certificate authority digital signature.

* Generate session keys in order to use symmetric encryption after the handshake is complete.

Steps of a TLS handshake

* TLS handshakes are a series of datagrams or messages exchanged by a client and a server.

* The actual steps depend on the key exchange algorithm. RSA is the most common one and it goes as follows...
  * Client initiates a message that has TLS version the client supports, cipher suites supported, a string of random bytes known as the "client random"

  * Server replies with the servers SSL certificate, the servers chosen cipher suite, server random.

* The client verifies the server's SSL certificate with the certificate authority that issues it.

* Client sends a `premaster secret` a random string encrypted with the servers public key that client gets from the certificate. This can only be decrypted by the server with the private key.

* Session key is created from the client random, the server random and the premaster secret. They should arrive at the same result (Deffie Hellman).

* Client sends a finished message that is encrypted with a session key.

* Server sends a finished message encrypted with the session key.

* Secure symmetric encryption is achieved.

### How Deffie-Hellman works

* Client sends hello the protocol version and the client random and a list of cipher suites.

* Server replies with SSL certificate, the chosen cipher and the server random. In this message server includes the server digital signature. Digital signature is the server using its private key to encrypt the client random, the server random, and the DH parameter. This encrypted data functions as the servers digital signature, establishing that the server had the private key that matches with the public key from the SSL certificate.

* The client decrypts the servers digital signature with the public key, verifying that the server controls the private key and is who it says it is. The client sends the client DH parameter to the server.

* Client and server calculate the premaster secret.  Instead of client generating premaster as in RSA client gives the DH parameter which his used to calculate the secret.

* Now the client and server calculate session keys from the premaster secret, client random, and server random just like in RSA handshake.
