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
 GRE is a tunneling protocol that encapsulates various network layer protocols within an IP header, establishing a logical, point-to-point connection between two routers over an existing IP infrastructure. This enables routing and connectivity between disparate networks as if they were directly linked, often serving as a foundation for more complex VPN solutions.

## Lab Topology

![Lab Topology](https://res.cloudinary.com/dkhycupho/image/upload/GRE_TUNNEL_Topology_jlaeyb.png)


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

![C1 interface config](https://res.cloudinary.com/dkhycupho/image/upload/c1_interface_cofig_illrb5.png)
![C1 DHCP Pool for Ubuntu client](https://res.cloudinary.com/dkhycupho/image/upload/C1_DHCP_POOL_config_for_Ubuntu_client_fzjeiu.png)
![C1 Routing table and ping success to C2](https://res.cloudinary.com/dkhycupho/image/upload/C1_routing_table_and_ping_success_to_C2_fakriu.png)
![C1 Tunnel interface config](https://res.cloudinary.com/dkhycupho/image/upload/Tunnel_interface_config_on_C1_lezjqq.png)
![C1 Tunnel interface success pings to C2 Tunnel interface](https://res.cloudinary.com/dkhycupho/image/upload/successful_pings_between_C1_tunnel_and_C2_Tunnel_fqub2u.png)
![C1 route advertisement](https://res.cloudinary.com/dkhycupho/image/upload/C1_ospf_config_to_advertise_route_to_C2_gtwglr.png)
![C1 Routing Table after Route advertisement](https://res.cloudinary.com/dkhycupho/image/upload/c1_Routing_Table_after_Router_advertisement_vq56o0.png)

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

![C2 interface config](https://res.cloudinary.com/dkhycupho/image/upload/C2_interface_config_lncly8.png)
![C2 DHCP Pool for Ubuntu client](https://res.cloudinary.com/dkhycupho/image/upload/C2_DHCP_POOL_config_for_unbuntu_client_th9cx3.png)
![C2 ping success to C1](https://res.cloudinary.com/dkhycupho/image/upload/C2_success_ping_to_C1_pmvjo0.png)
![C2 Tunnel interface config](https://res.cloudinary.com/dkhycupho/image/upload/Tunnel_Interface_config_on_C2_msorpr.png)
![C2 Tunnel interface success pings to C1 Tunnel interface](https://res.cloudinary.com/dkhycupho/image/upload/successful_pings_between_C2_tunnel_and_C1_tunnel_qnyllv.png)
![C2 Ospf advertised route to C1](https://res.cloudinary.com/dkhycupho/image/upload/C2_ospf__advertised_route_to_C1_jaj4xy.png)
![C2 Routing Table after Route advertisement](https://res.cloudinary.com/dkhycupho/image/upload/C2_Routing_Table_after_Route_advertisement_yidzom.png)


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

![ISP1 Routing Table](https://res.cloudinary.com/dkhycupho/image/upload/ISP1_Routing_table_to_show_its_not_learning_the_Ip_add_of_the_tunnel_interface_untbmm.png)
![ISP2 Routing Table](https://res.cloudinary.com/dkhycupho/image/upload/ISP2_Routing_table_to_show_its_not_learning_the_Ip_add_of_the_tunnel_interface_s7mwmi.png)
![ISP3 Routing Table](https://res.cloudinary.com/dkhycupho/image/upload/ISP3_Routing_table_to_show_its_not_learning_the_Ip_add_of_the_tunnel_interface_lqdwma.png)

## Snapshot from wireshark showing the GRE being encapsulated in the IP
![Wireshark IP inspection](https://res.cloudinary.com/dkhycupho/image/upload/wireshark_showing_the_src_and_dest_ip_used_while_transferring_the_packet_cestym.png)
![Wireshark IP inspection](https://res.cloudinary.com/dkhycupho/image/upload/v1750683436/Wireshark_shows_the_GRE_ip_being_encapsulated_and_not_known_by_the_ISP_routers_hkajjf.png)

## Snapshot of pings between Ubuntu clients on the 2 customer sites
![pings from Ubuntu Client at C1 to Ubuntu client at C2](https://res.cloudinary.com/dkhycupho/image/upload/Succesful_pings_from_Ubuntu_client_at_C1_site_to_the_ubuntu_Client_at_C2_site_tkmpcb.png)
![pings from Ubuntu Client at C2 to Ubuntu client at C1](https://res.cloudinary.com/dkhycupho/image/upload/Succesful_pings_from_Ubuntu_client_at_C2_site_to_the_ubuntu_Client_at_C1_site_by5mol.png)

### Conclusion
This lab provided a practical walkthrough for configuring and verifying a GRE tunnel in a GNS3 environment. We achieved the primary objective of establishing routable connectivity between two remote networks across an intermediary "ISP" network using GRE encapsulation. Verification steps, including ping and traceroute, confirmed the tunnel's functionality.

### Future Work:
To further enhance this lab and deepen understanding, the following could be explored:
* **IPsec Integration:** Implement IPsec over the GRE tunnel to add encryption and authentication, transforming it into a secure VPN solution.
* **Dynamic Routing:** Configure a dynamic routing protocol (e.g., OSPF or EIGRP) to advertise routes across the GRE tunnel, demonstrating automatic route discovery.
* **Advanced Tunneling:** Investigate other tunneling protocols like DMVPN or VTI for more scalable and flexible VPN designs.
