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

![Lab Topology](Images/GRE_TUNNEL_Topology.png)


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

![C1 interface config](Images/c1_interface_config.png)
![C1 DHCP Pool for Ubuntu client](Images/C1_DHCP_POOL_config_for_Ubuntu_client.png)
![C1 Routing table and ping success to C2](Images/C1_routing_table_and_ping_success_to_C2.png)
![C1 Tunnel interface config](Images/Tunnel_interface_config_on_C1.png)
![C1 Tunnel interface success pings to C2 Tunnel interface](Images/successful_pings_between_C1_tunnel_and_C2_Tunnel.png)
![C1 route advertisement](Images/C1_ospf_config_to_advertise_route_to_C2.png)
![C1 Routing Table after Route advertisement](Images/c1_Routing_Table_after_Router_advertisement.png)

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

![C2 interface config](Images/c2_interface_config.png)
![C2 DHCP Pool for Ubuntu client](Images/C2_DHCP_POOL_config_for_Ubuntu_client.png)
![C2 ping success to C1](Images/C2_success_ping_to_C1.png)
![C2 Tunnel interface config](Images/Tunnel_interface_config_on_C2.png)
![C2 Tunnel interface success pings to C1 Tunnel interface](Images/successful_pings_between_C2_tunnel_and_C1_tunnel.png)
![C2 Ospf advertised route to C1](Images/C2_ospf_advertised_route_to_C1.png)
![C2 Routing Table after Route advertisement](Images/C2_Routing_Table_after_Route_advertisement.png)


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

### VERIFICATIONS

## ISPs Routing Table not learning anything about the Tunnel Ip Address

![ISP1 Routing Table](Images/ISP1_Routing_table_to_show_its_not_learning_the_Ip_add_of_the_tunnel_interface.png)
![ISP2 Routing Table](Images/ISP2_Routing_table_to_show_its_not_learning_the_Ip_add_of_the_tunnel_interface.png)

## Snapshot from wireshark showing the GRE being encapsulated in the IP
![Wireshark IP inspection](Images/wireshark_showing_the_src_and_dest_ip_used_while_transferring_the_packet.png)
![Wireshark IP inspection](Images/Wireshark_shows_the_GRE_ip_being_encapsulated_and_not_known_by_the_ISP_routers.png)

## Snapshot of pings between Ubuntu clients on the 2 customer sites
![pings from Ubuntu Client at C1 to Ubuntu client at C2](Images/Succesful_pings_from_Ubuntu_client_at_C1_site_to_the_ubuntu_Client_at_C2_site.png)
![pings from Ubuntu Client at C2 to Ubuntu client at C1](Images/Succesful_pings_from_Ubuntu_client_at_C2_site_to_the_ubuntu_Client_at_C1_site.png)

### Conclusion
This lab provided a practical walkthrough for configuring and verifying a GRE tunnel in a GNS3 environment. We achieved the primary objective of establishing routable connectivity between two remote networks across an intermediary "ISP" network using GRE encapsulation. Verification steps, including ping and traceroute, confirmed the tunnel's functionality.

### Future Work:
To further enhance this lab and deepen understanding, the following could be explored:
* **IPsec Integration:** Implement IPsec over the GRE tunnel to add encryption and authentication, transforming it into a secure VPN solution.
* **Dynamic Routing:** Configure a dynamic routing protocol (e.g., OSPF or EIGRP) to advertise routes across the GRE tunnel, demonstrating automatic route discovery.
* **Advanced Tunneling:** Investigate other tunneling protocols like DMVPN or VTI for more scalable and flexible VPN designs.
