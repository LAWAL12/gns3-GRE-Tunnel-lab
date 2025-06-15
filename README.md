# GNS3 Lab: GRE Tunnel Implementation

## Table of Contents
- [Overview](#overview)
- [Lab Topology](#lab-topology)
- [Objective](#objective)
- [Configuration Steps](#configuration-steps)
- [Verification](#verification)
- [Conclusion](#conclusion)

## Overview
This repository documents a GNS3 lab demonstrating the configuration and verification of a Generic Routing Encapsulation (GRE) tunnel. The lab establishes a secure and routable path between two remote networks over an insecure public network.

## Lab Topology

![Lab Topology](Images/GRE TUNNEL Topology.png)


*(In my Lab I made use of "Two Cisco IOSv routers (C1, C2) connected via three Cisco IOSV routers acting as an 'ISP' cloud. Each router(C1 and C2) has a LAN segment behind it with an Ubuntu client connected to each router (c1 and C2).")*

## Objective
The primary objectives of this lab were:
1.  Configure basic IP addressing on all devices.
2.  Establish a GRE tunnel between Customer site1(C1) and Customer site2(C2).
3.  Configure routing (e.g., static routes or OSPF/EIGRP) over the GRE tunnel.
4.  Verify end-to-end connectivity through the tunnel.

## Configuration Steps

### C1 Configuration
c1(config-if)#int g0/1      
c1(config-if)#ip add dhcp
c1(config-if)#no sh
c1(config-if)#int g0/0 
c1(config-if)#ip add 10.1.1.1 255.255.255.0
c1(config-if)#no sh

c1#conf t
c1(config)#ip dhcp pool CS1_pool
c1(dhcp-config)#network 10.1.1.0 255.255.255.0
c1(dhcp-config)#default-router 10.1.1.1


c1(config)#int tunnel 0
c1(config-if)#ip add 10.1.3.1 255.255.255.0
c1(config-if)#tunnel source g0/1
c1(config-if)#tunnel destination 8.8.11.4

c1(config)#router ospf 100
c1(config-router)#network 10.1.0.0 0.0.255.255 area 0

## C1 Router Snipshots

![C1 interface config](Images/c1 interface config.png)
![C1 DHCP Pool for Ubuntu client](Images/C1 DHCP POOL config for Ubuntu client.png)
![C1 Routing table and ping success to C2](Images/C1 routing table and ping success to C2.png)
![C1 Tunnel interface config](Images/Tunnel interface config on C1.png)
![C1 Tunnel interface success pings to C2 Tunnel interface](Images/successful pings between C1 tunnel and C2 Tunnel.png)
![C1 route advertisement](Images/C1 ospf config to advertise route to C2.png)
![C1 Routing Table after Route advertisement](Images/c1 Routing Table after Router advertisement.png)

### C2 Configuration
c2(config-if)#int g0/1      
c2(config-if)#ip add dhcp
c2(config-if)#no sh
c2(config-if)#int g0/0
c2(config-if)#ip add 10.1.2.1 255.255.255.0
c2(config-if)#no sh

c2#conf t
c2(config)#ip dhcp pool CS1_pool
c2(dhcp-config)#network 10.1.2.0 255.255.255.0
c2(dhcp-config)#default-router 10.1.2.1


c2(config)#int tunnel 0
c2(config-if)#ip add 10.1.3.2 255.255.255.0
c2(config-if)#tunnel source g0/1
c2(config-if)#tunnel destination 8.8.10.4

c2(config)#router ospf 100
c2(config-router)#network 10.1.0.0 0.0.255.255 area 0


## C2 Router Snipshots

![C2 interface config](Images/c2 interface config.png)
![C2 DHCP Pool for Ubuntu client](Images/C2 DHCP POOL config for Ubuntu client.png)
![C2 ping success to C1](Images/C2 success ping to C1.png)
![C2 Tunnel interface config](Images/Tunnel interface config on C2.png)
![C2 Tunnel interface success pings to C1 Tunnel interface](Images/successful pings between C2 tunnel and C1 tunnel.png)
![C2 Ospf advertised route to C1](Images/C2 ospf advertised route to C1.png)
![C2 Routing Table after Route advertisement](Images/C2 Routing Table after Route advertisement.png)


### ISP1 Configuration
ISP1(config)#int g0/0
ISP1(config-if)#ip address 8.8.10.1 255.255.255.0
ISP1(config-if)#no sh
ISP1(config)#int g0/1
ISP1(config-if)#ip address 8.8.8.2 255.255.255.0

ISP1(config-if)#no sh

ISP1(config)#ip dhcp pool customers
ISP1(dhcp-config)#network 8.8.10.0 255.255.255.0
ISP1(dhcp-config)#default-router 8.8.10.1
ISP1(dhcp-config)#dns-server 8.8.8.1


ISP1(config)#router ospf 1
ISP1(config-router)#network 8.8.8.2 0.0.0.0 area 0

ISP1(config)#router bgp 65000
ISP1(config)#neighbor 8.8.8.1 remote-as 65000
ISP1(config)#neighbor 8.8.9.2 remote-as 65000

### ISP2 Configuration
ISP2(config)#int g0/0
ISP2(config-if)#ip address 8.8.8.1 255.255.255.0
ISP2(config-if)#no sh
ISP2(config)#int g0/1
ISP2(config-if)#ip address 8.8.9.1 255.255.255.0
ISP2(config-if)#no sh

ISP2(config)#router ospf 1
ISP2(config-router)#network 0.0.0.0 0.0.0.0 area 0

ISP2(config)#router bgp 65000
ISP2(config)#neighbor 8.8.8.2 remote-as 65000
ISP2(config)#neighbor 8.8.9.2 remote-as 65000

### ISP3 Configuration
ISP3(config)#int g0/0
ISP3(config-if)#ip address 8.8.9.2 255.255.255.0
ISP3(config-if)#no sh
ISP3(config)#int g0/1
ISP3(config-if)#ip address 8.8.11.1 255.255.255.0
ISP3(config-if)#no sh

ISP3(config)#ip dhcp pool customers
ISP3(dhcp-config)#network 8.8.11.0 255.255.255.0
ISP3(dhcp-config)#default-router 8.8.11.1 
ISP3(dhcp-config)#dns-server 8.8.8.1 

ISP3(config)#router ospf 1
ISP3(config-router)#network 8.8.9.2 0.0.0.0 area 0

ISP3(config)#router bgp 65000
ISP3(config)#neighbor 8.8.8.2 remote-as 65000
ISP3(config)#neighbor 8.8.9.1 remote-as 65000


## ISPs Routing Table not learning anything about the Tunnel Ip Address

![ISP1 Routing Table](Images/ISP1 Routing table to show its not learning the Ip add of the tunnel interface.png)
![ISP2 Routing Table](Images/ISP2 Routing table to show its not learning the Ip add of the tunnel interface.png)

## Snapshot from wireshark showing the GRE being encapsulated in the IP
![Wireshark IP inspection](Images/wireshark showing the src and dest ip used while transferring the packet.png)
![Wireshark IP inspection](Images/Wireshark shows the GRE ip beign encapsulated and not known by the ISP routers.png)


