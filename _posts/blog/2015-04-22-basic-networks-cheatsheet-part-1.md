---
layout: post
title: 'Basic Networks: Cheatsheet [Part 1]'
date: 2015-04-22 21:54:00.000000000
type: post
published: true
status: publish
excerpt: 
    The purpose of this cheat-sheet is to briefly overview the main networking principles 
    without going into too much technical details...
categories:
- Networking
tags:
- BGP
- Network
- OSPF
- RIP
- NAT
- Cheatsheet
---

Present day distributed systems are based on technologies which abstract away the complexity of the underlying network. 
Software Engineers use middleware and programing APIs which completely mask how the underlying network works. 
Although this eases software development, programmers quite often have problems troubleshooting subtle performance 
quirks as they do not understand what happens “below” the programming abstractions.

The purpose of this cheat-sheet is to briefly overview the main networking principles without going into too much 
technical details. For more in-depth understanding you can refer to 
[Tanenbaum’s bible](http://www.amazon.com/Computer-Networks-Edition-Andrew-Tanenbaum/dp/0132126958).

# Routing

## Basic Routing Theory

The Internet consists of a number of [Autonomous Systems](http://en.wikipedia.org/wiki/Autonomous_system_%28Internet%29) (AS), 
which in term consist of one or multiple subnetworks. Typically, an AS is owned and exploited by a single 
large-scale organisation – e.g. an ISP. Routing protocols enable hosts to communicate efficiently regardless 
of which network they are in.

Routing protocols can be divided in two groups with respect to whether they route traffic within or across AS:

*   Interior Gateway Protocols – route within an AS;
*   Exterior Gateway Protocols – route across AS networks.

With respect to the used routing technique, protocols can also be divided into:

*   Distance Vector protocols – each router maintains a lookup table mapping destination addresses with 
the most suitable “next step” in the routing. Routers exchange data only with their adjacent routers, 
which update their tables accordingly. Usually based on the 
[Ford-Bellman algorithm](http://en.wikipedia.org/wiki/Bellman%E2%80%93Ford_algorithm).
*   Link State protocols – each router “knows” the states of all links in the network and routes accordingly. 
Changes are broadcasted to all routers. Based on [Dijkstra’s algorithm](http://en.wikipedia.org/wiki/Dijkstra%27s_algorithm).

The above routing methods are generic and can use any distance measure – e.g. number of hops or latency. 
Typically, distance vector protocols scale better as they do not flood the entire network with network state data.

Finally, routing protocols can also be classified as:

*   Classful – when routers do not include subnet masks in their messages. 
Routers use the [legacy IP classes](http://en.wikipedia.org/wiki/Classful_network) to determine subnet and host addresses.
*   Classless - when routers exchange subnet information together with the destination addresses.

Modern protocols tend to be classless.

Routing is typically hierarchical, meaning that each network can be divided into subnetworks. 
Each subnet has two types of routers – internal and border. Internal routers route traffic within the network, 
while border routers route to and from other networks from the same level in the hierarchy.

Network devices (like routers) typically communicate through 
[TCP](http://en.wikipedia.org/wiki/Transmission_Control_Protocol) or 
[UDP](http://en.wikipedia.org/wiki/User_Datagram_Protocol). In case something goes wrong, 
network admins need a way to “debug” what has happened. Enter Internet Control Message Protocol (ICMP) 
which is used just for that. One notable example is the *ping* command, which uses ICMP to 
check the connection to a host.

## Large scale routing (Exterior Gateway Protocol)

Routing between Autonomous Systems (AS) is handled by the Border Gateway Protocol (BGP). 
It is pretty much the only Exterior Gateway Protocol in use these days. Each AS is given 
an Id number so it can be uniquely identified. Within each AS a few routers are designated 
to manage the traffic with other AS networks. They are known as “Autonomous System Boundary Routers” (ASBR). 
BGP is classified as a distance vector protocol. However, the designated BGP routers within each AS keep the 
entire path to each destination AS. Thus, for each destination the neighbouring routers exchange entire paths, 
not just the next best hop. BGP is classless.

## Smaller Scale routing (Interior Gateway Protocols)

Open Shortest Path First (OSPF) is a link state classless protocol used to route data within an AS. 
Each router maintains a complete weighted graph of the network and runs 
[Dijkstra’s algorithm](http://en.wikipedia.org/wiki/Dijkstra%27s_algorithm) to determine the path 
for each packet. Each router’s graph is built by exchanging graphs with the adjacent routers or via 
multicast if the network supports that.

OSPF can manage big networks (an entire AS) and thus is hierarchical and can consist of smaller 
networks called areas. Each area has an Area Border Router (ABR) which represents it in the 
OSPF routing. The internal structure of area is not known to the OSPF routers.

The Intermediate System to Intermediate System (IS-IS) protocol is conceptually very similar OSPF, 
the main difference being that it does 
[not use IP addresses](http://en.wikipedia.org/wiki/IS-IS#Comparison_with_OSPF). 
This is why it is less widely adopted.

The Routing Information Protocol (RIP) can also be used to route within an AS, 
but it does not scale as well as OSPF. It is a distance vector protocol and distances are 
exchanged periodically (usually every 30 seconds). Its first version was classful, but 
the later ones are classless.

<figure>
  <img src="/images/blog/Basic Networks Cheatsheet Part 1/as.jpg" alt="Autonomous Systems" >
  <figcaption>Autonomous Systems.</figcaption>
</figure>


# Network Addresses

To function networks need to give each host a unique address. Usually this is the notorious 
IPv4 address. Because of hierarchical routing (i.e. networks are divided into subnets), each IP 
address is divided in two parts – the network id and the host id.

Routing is performed based on the network address (i.e. from one network into another) until the 
destination network is achieved. At this point the respective router can directly send the package 
to the host, using the host portion of the IP address.

Back in the day IP addresses were [divided into classes](http://en.wikipedia.org/wiki/Classful_network). 
Each class can be recognised by a specific sequence of bits in the address’ start. Each class would define 
which part of the address is a subnet mask and which is the host address. This mechanism turned out to be 
wasteful and was eventually deprecated.

Presently arbitrary subnet address are used (i.e. Classless Inter-Domain Routing, CIDR), and this can be 
ncluded in the IPv4 notation as well. For example 152.121.183.34<u>/21</u> means that the first 21 bits 
are the network address, and the rest are the host address.

Network routing works with IP addresses, but local area networks (like Ethernet or WiFi) work with physical 
Media Access Control (MAC) addresses. A MAC address is 6 bytes and can be recorded in one of the following 
formats: <u>01-23-45-67-89-ab</u>; <u>01:23:45:67:89:ab</u> or <u>0123.4567.89ab</u>. So we need to map between 
MAC and IP addresses. Enter the Address Resolution Protocol (ARP). Each host in the LAN keeps a dictionary of 
MACs and IPs in the network. Whenever an unknown IP address comes in, a special ARP message with the IP is 
broadcasted. The machine with the IP in question responds in order to notify the rest of the hosts to update 
their dictionaries. The [Neighbor Discovery Protocol](http://en.wikipedia.org/wiki/Neighbor_Discovery_Protocol) (NDP) 
is similar to ARP but it works with IPv6.

The IPv4 address space can have only 2<sup>32</sup> (4,294,967,296) addresses, which obviously is insufficient 
for all devices connecting to the Internet. One way to solve this problem is not to assign a static IP address to 
every connected device – i.e. to reuse a pool of IP addresses and dynamically assign them to devices joining the network. 
Also, static IP addresses are problematic when a device is moved from one network to another. Enter the Dynamic Host 
Configuration Protocol (DHCP). Once a new machine joins a network it broadcast a request to find the dedicated DHCP server, 
which responds with its IP address. If there is no DHCP address on the local network, one of the hosts on the network 
(known as DHCP agent) transmits the messages between the newly added machine and an external DHCP server.

Another way to mitigate the shortage of IPv4 address is to use Network Address Translation (NAT). In this case, 
every device in a subnet is given a “hidden” IP address, which may not be globally unique. Usually the private 
IP addresses are in the forma 192.168.X.X or 10.X.X.X. These IP ranges are “reserved” for private (e.g. NAT-ed) 
networks and can not be used in the public internet. All devices’ communication passes through a NAT device 
(e.g. a router) which overwrites the source IP address with its own and changes the outbound port uniquely, so 
that when data is received back on this port, the NAT device knows who the recipient is. As a consequence, a server 
outside the NAT-ed network can not start communication with a machine behind NAT, unless the machine has started 
it first. Otherwise, the external server would not know which port to send to.

<figure>
  <img src="/images/blog/Basic Networks Cheatsheet Part 1/nat.jpg" alt="NAT" >
  <figcaption>NAT.</figcaption>
</figure>

NAT is basically a hack to work around the issue of limited IPv4 addresses.
A better approach would be to move to IPv6, which would allow all devices to have a static IPs. 
IPv6 addresses are 16 bytes (128 bits) and are in the form: 2001:0db8:85a3:0042:1000:8a2e:0370:7334.

Resources:

*   [https://www.youtube.com/user/technologyaddict405/videos](https://www.youtube.com/user/technologyaddict405/videos)
*   [http://en.wikibooks.org/wiki/Communication_Networks/Routing](http://en.wikibooks.org/wiki/Communication_Networks/Routing)
