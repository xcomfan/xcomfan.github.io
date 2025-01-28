---
layout: page
title: "Adrian Cantrill Tech Fundamentals Course Notes"
permalink: /unsorted/tech_fundamentals
---
#UNPROCESSED

# OSI 7-Layer Network Model

## Layer 1 - Physical Layer

Layer 1 (Physical) specifications define the transmission and reception of **raw bit stream** between devices and a **shared** physical medium. It defines things like voltage levels, timing, rates, distances,  modulation and connections. For a copper cable electrical signals are used, for fibre light is used, WiFi uses RF. The standards make it such that each device can understand the medium being used to receive the binary signal.

At layer 1, there is not direct device addressing. One computer cannot directly address another computer. Even if you are suing a hub anything ent by one computer is transmitted to all of the computers.

Only one device can transmit at once on a shared medium (cable or something similar linking the devices) in Layer 1. If two devices transmit at once there is a collision and it will corrupt any transmission on the shared medium. At layer 1 there is no method of controlling which devices can transmit so collisions are almost guaranteed.

Effectively there IS NO device to device communication on a Layer 1 network. Everything is broadcast to the shared medium.

## Layer 2 - Data Link Layer

Layer 2 is built on top of Layer 1. Layer 2 network requires a functional Layer 1 network to operate. It does not matter if Layer 1 is copper fibre or something else Layer 2 will have same capabilities.

Layer 2 introduces the concept of a Data Link Frame and a MAC address for every devices on the network.

A MAC address is 48 bits in hex. 24 bits is used for the manufacturer of the networking hardware and the rest is used to make the device unique.

Layer 2 provides frames and Layer 1 provides the physical medium for sending those Frames.

A Data Frame is a containers of sorts. First part of a frame is the Preamble and start frame delimiter. This lets devices know where a Frame starts so they can know where to look for the various components of the Frame.  Next come the destination and source MAC addresses. In Layer 2 you can send a message to a specific device by putting its MAC address in the destination of use all `F` in MAC address to send to all devices (this is known as a broadcast). Next is the "Ether Type" which is commonly used to specify which Layer 3 protocol is putting data inside the Frame. Next comes the payload which is the actual data that is being carried by the Frame. This data is a Layer 3 IP Packet. Last you have the CRC check to help identify any errors in the Frame.

| Preamble 56 bits SFD - 8 Bits | Destination MAC address | Source MAC Address | ET 16 Bits | Payload 46 - 1500 Bytes | FCS Check 32 bits |
| ----------------------------- | ----------------------- | ------------------ | ---------- | ----------------------- | ----------------- |

Layer 2 also provides controlled access to the Physical Medium which addresses the collision problem of Layer 1. Layer 2 can communicate with Layer 1 and look for signs of a carrier signal. If any device is transmitting at time of check you would see the carrier signal on the Layer 1 network. This is done via CSMA (Carrier Sense Multiple Access). So Layer 2 waits for Layer 1 to be free (nothing transmitting) to send its data. If two devices find no carrier signal and send at the exact same time a collision is detected via Collision Detections (CD) then a jam signal is sent by all devices which detect it an a random backoff occurs. CD is a part of Layer 2 and part of Layer 2 CSMA/CD. Random backoff is each device backing off for a random time and trying to send again. IF collision is detected again they will back off for a longer random time. Essentially you are reducing likelihood of collision. Collision detection is what allows multiple devices to coexist on the same network. Collision detecting works by the sending device checking for another device transmitting while its sending. If detected it sends the Jam signal.

When data is received on Layer 1 and given to Layer to that is called de-encapsulated and encapsulation is when you pass the data to a lower Layer from 2 to 1 for example. You encapsulate the data into a Frame and de-encapsulate it from the Frame.

One thing of note is Layer 2 will talk to Layer 2 and they don't really care about Layer 1. Layer 2 abstracts away the Level 1 details.

Revisiting using a HUB. A HUB is a layer 1 device so it will jure rebroadcast all Layer 1 traffic including Collisions. A switch however is a Layer 2 devices. Switch maintains a MAC address table whit it uses to learn what is connected to each of its port. Thus a switch can be smart enough to take the traffic from one of the machines connected to it, and send it directly to another machine connected to it (while switch is populating the MAC address table it acts like a HUB. In other words if it does not know which port destination MAC address is on it will sent to all the ports). This also makes it so that each of the ports on a Switch is its own collision domain. Only collisions between the switch port and device its connected to can occur on that domain. Corrupted data will not be forwarded to any other ports.

The internet is a collection of Layer 2 networks.

Layer 2 allows for identifiable devices. Media access control and collision detection. It also allows you to do unicast communication (1:1) and multicast communication (one to many).

## Layer 3 - Network Layer

Job of Layer 3 is to get data from one device to another. Think of loading a website fo example.

Layer 3 deals with the IP packet similar to how Layer 2 deals with Frames.

Every IP Packet has a source and destination IP address. There is also a protocol field which stores which Layer 4 protocol is being transmitted by this Layer 3 packet. This protocol fields specifies if you are using TCP (value will be 6), ICMP (value will be 1) or UDP (value will be 17). The rest is the data section as well as a TTL (which is used to control the maximum number of hops so that the packet will not bounce around forever.) There is a header section of the IP packet the captures the IP protocol version, header length, types of service, total length, identification, IP flags, and fragment offset.

Layer 3 works with both IPv4 and IPv6.

### IPv4 details

Lets use IP address 133.33.3.7 as an example.  IP addresses use dotted decimal notation and are made of 4 numbers between 0 and 255 separated by dots (`.`).

The first two numbers (133.33 in our example) are the network part of the address. The second two numbers (3.7 in our example are the host parts). If network part of two IP addresses matches then they are on the same IP network. If its different they are on different networks.

IP addresses can be assigned manually of via DHCP. IP addresses need to be unique on your local network or bad things happen.

Subnet Masks help us determine if a network is local. They are configured on network interfaces. Default Gateway is also configured on network interfaces. That is where packets are forwarded to if the destination is not a local IP address.

#### Submask Deep Dive

Lets take our example IP address and break it down into binary

`133.33.3.7` becomes:

`10000101.00100001.00000011.00000111`

When applying/creating a subnet mask a `1` represents a part of the address that is used for network and a `0` denotes a part that is used for hosts.

When we refer to a slash n network such as a slash 16 (`/16`) that is a count of ones. So a slash 16 network will have 16 ones `11111111.11111111.00000000.00000000`. So if you overlay an IP address and a subnet its easy to tell which is the network part and which is the host part. Network mask also makes it easy to calculate the start of a network.  If you use all the numbers in your IP address where the subnet has 1s and set all of the host portion to zeo that will be the start of the network addresses. If you use all of the numbers in your IP address where the subnet 1s and set the host portion to all 1s that will be the end of the range.

The higher the slash number the more specific the address. A `/32` is the maximum and represents just one IP address.

### Route Tables and Routes

Every router has a routing table per interface and in that routing table it has a Destination network as well as the next hop/target. For example.

| Destination    | Next Hop/Target |
| -------------- | --------------- |
| 52.217.13.0/24 | 52.217.13.1     |
| 0.0.0.0/0      | 52.42.214.1     |
| 52.43.215.0/24 | 52.43.215.1     |

If multiple routes in routing table match the most specific one is used. In above example 0.0.0.0/0 is most specific so its the default route.

Route tables can be populated statically or via BGP.

Routing is the process where packets are routed hop by hop from source to destination.

MAC address of the destination routed is done via ARP (Address Resolution Protocol)

### ARP

ARP is the protocol used to find a MAC address for a given IP address. 

As an example lets say we have two computers one at IP `133.33.3.7` and one at `133.33.3.10` trying to send data. At Layer 3 we create a packet with `133.33.3.7` as the source and `133.33.3.10` as the destination. When we encapsulate this packet into Layer 2 we need to know the MAC address. ARP will send a broadcast on all FFF address asking for who has the  `133.33.3.10`. The computers with the `133.33.3.10` address will respond with its MAC address. Now that the sending computer has the MAC address is can encapsulate the Frame to Layer 1 and send the data over the physical medium.

In a more complex example where the two computers are not on the same network routers get involved. The sending computer will check the destination IP address and use the Netmask to determine if its sending to local network or not. Since its not a local network send the default gateway IP is used which is the router. So at layer 3 we have the destination IP of the system we are sending to, but at Layer 2 we are going to use ARP and find the MAC address of the Router to send it the Frame. When the router receives the packet will see that the MAC address is its own and it will strip away the frame leaving just the packet. A normal device such as a computer or phone will just drop a packet not intended for it, but a router is meant to route packets. At this point the router will check the destination IP of the packet and use its routing table to determine  where to send it for the next hop. It will encapsulate the packet into a Fram and use ARP to find the MAC address for the next hop target (next router on the network). This goes on until we reach the router where the destination computer is local.

### Layer 3 Summary

Layer 3 adds IPv4 and IPv6 also known as cross network addressing

It also adds ARP for finding the MAC address of an IP

Layer 3 adds routes which define where to forward this packet, and route tables so that multiple routes can be managed.

Routers are Layer 3 devices. They move packets from source to destination encapsulating in L2 along the way.

Layer 3 allows for device to device communication over the internet.

Layer 3 DOES NOT provide a method for channels for communication.  You only have packets with source and destination. You can't have different applications on two computers communicating with each other at the same time. This is addressed by layers 4 and 5. Layer 3 just gives you one stream of data.

Layer 3 packets may also be delivered out of order. Due to network conditions packets may arrive in different order then they were sent in.

## Layer 4 and 5 Transport and Session Layer

Mixing these two because its easier to cover them together as there is some overlap between the two. "Its generally not worth the argument to decide if something is covered in Layer 4 or Layer 5"

Layer 3 Limitations:

* At Layer 3 each packet is an independent entity and is routed separately over the internet. This leads to the potential issues of:
	* Out of order arrival of packets
	* Packets can just go missing due to network conditions. (Not Guaranteed to be reliable)
	* Per packet routing can introduce delays to packets in route. Different packets can experience different delays.
	* No way to separate the packets meant for different applications.
	* No flow control. If source is sending faster than destination can receive it can saturate the destination causing packet loss.


### Layer 4

Layer 4 adds two protocols. TCP (Transmission Control Protocol) and UDP (User Datagram Protocol). Both work on top of IP and have a different set of features.

#### UDP

Faster than TCP but less reliable. It does not have the overhead needed for reliable delivery which TCP does.
#### TCP

Provides reliability, error handling and ordering of data. Used for HTTP, HTTPS, SSH and so on.

TCP is a connection oriented protocol and establishes a bi directional channel of communications between two devices.

TCP operates on **segments**. TCP segments are encapsulated within IP packets. Segments don't have source and destination IP addresses, the packets provide IP addressing.

Segment has the following structure. ***Note:*** options and padding is omitted for simplicity.

| Source Port | Destination Port | Sequence Number | Acknowledgement | Flags and things* | Window | Checksum | Urgent Pointer | Data | 

Ports allow for multiple applications on same computer to communicate at the same time.

Sequence Number is used to determine order of packets and can also be used in error correction. It is incremented with each segment sent.

Acknowledgment is used for one side to indicate that it received up to and including a certain sequence number. Every segment that is transmitted needs to be acknowledged.

Flags and things is used for various controls over TCP segments and the wider connection. Can close the connection here, but also have options for offsets etc. Some of the value are `FIN` to close connection `ACK` for acknowledgements and `SYN` for synchronizing sequence numbers.

Window indicates the number of bytes you are willing to receive between acknowledgements. Sender will pause until you receive that amount of data. This is used for flow control.

Checksum used for error checking.

Urgent Pointer is used to indicate high priority traffic. For example if 99% of traffic send is just data and 1% is control traffic you can give the control traffic priority. FTP and Telnet use this field for example.

All of these above are in the TCP header. The remainder of the segment is the actual data being sent. 

TCP provides for a way to send a reliable (segment can be retransmitted if it fails to arrive) ordered (we have sequence numbers) set of data.  You can build on this to establish a connection between two systems.

##### TCP 3 Way Handshake
Step 1: Client sends a segment with `SYN` in the flags and things part of the segment with a random sequence number known as `CS`. The random sequence number is called ISN (Initial Sequence Number) This is how the client says it wants to establish a connection.

Step 2: Server replies with a `SYN-ACK` segment. It picks its own random sequence number known as `SS` (Server Sequence) It sets the acknowledgement part to `CS+1` from the client call. This is saying I have received up to `CS` now send me `CS + 1` . 

SYN is synchronize sequence numbers and SYN-ACK is also used to synchronize sequence numbers (its the acknowledgment)

Step 3: Client sends segment with `ACK` (Acknowledge) set to `SS+1` which means I have received `SS` now send me `SS+1` and the sequence set to `CS + 1`

At this point the connection is established. Going forward any time either side sends data it increments the sequence and the other side acknowledges the sequence + 1. This allows for re-transmission when data is lost.

##### Sessions and State
Strictly speaking the concept of a session and an established connection is considered Layer 5

# Network Address Translation (NAT)

NAT translates private IPv4 addresses to public ones.

NAT is designed to overcome IPv4 shortages

## Static NAT
**Static NAT** maps one private IP to one fixed public address. **Dynamic NAT** is similar but you have a pool of public IP addresses. So its a one private IP maps to first available public IP. Typically used when you have a lot of private IP addresses that need access to the internet.

#TODO: need to write up and link public vs private addresses.

Private IP addresses cannot communicate over the public internet. 

With Static NAT the router (NAT Device) maintains a NAT table which maps private IP to public IP in a one to one mapping. So each device has a private IP and a public one assigned to it in the NAT table.

If trying to communicate to a public IP the client will generate a packet with its private IP as the source and the public IP as the destination. Since the destination is not local it will be sent to the Default Gateway which is the router. As the packet passed though the route the source address will be swapped by the route (the NAT device here) based on the NAT table.  (AWS Internet Gateway does this FYI). On the way back the same thing happens, but here the NAT device swaps the public IP in the destination for the private one so that the requesting client can get the response from the public IP.

**Port Address Translation (PAT)** is when you have many private devices mapping to one public IP address. This is what most home routers do.

NAT is only needed for IPv4. IPv6 has so many addresses that translation is not needed.













