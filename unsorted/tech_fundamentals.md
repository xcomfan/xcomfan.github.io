---
layout: page
title: "Adrian Cantrill Tech Fundamentals Course Notes"
permalink: /unsorted/tech_fundamentals
---

## OSI 7-Layer Network Model

### Layer 1 - Physical Layer

Layer 1 (Physical) specifications define the transmission and reception of **raw bit stream** between devices and a **shared** physical medium. It defines things like voltage levels, timing, rates, distances,  modulation and connections. For a copper cable electrical signals are used, for fibre light is used, WiFi uses RF. The standards make it such that each device can understand the medium being used to receive the binary signal.

At layer 1, there is not direct device addressing. One computer cannot directly address another computer. Even if you are suing a hub anything ent by one computer is transmitted to all of the computers.

Only one device can transmit at once on a shared medium (cable or something similar linking the devices) in Layer 1. If two devices transmit at once there is a collision and it will corrupt any transmission on the shared medium. At layer 1 there is no method of controlling which devices can transmit so collisions are almost guaranteed.

Effectively there IS NO device to device communication on a Layer 1 network. Everything is broadcast to the shared medium.

### Layer 2 - Data Link Layer

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

| Destination | Next Hop/Target |
| ----------- | --------------- |
| 52.217.13.0/24 | 52.217.13.1 |
| 0.0.0.0/0 | 52.42.214.1 |
| 52.43.215.0/24 | 52.43.215.1 |

If multiple routes in routing table match the most specific one is used. In above example 0.0.0.0/0 is most specific so its the default route.

Route tables can be populated statically or via BGP.

Routing is the process where packets are routed hop by hop from source to destination.

MAC address of the destination routed is done via ARP (Address Resolution Protocol)