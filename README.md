
<br>

# **Enterprise Network**

<br>

## **Introduction and Objectives**


This project presents an advanced Cisco-based enterprise network built in GNS3 as part of a study portfolio. 
The network simulates a real organization with a central Headquarters site and a remote Branch site, using a realistic Layer 2 and Layer 3 design based on the OSI model and extended to application-level services.

The main objective is to show consistent routing, services, and management across multiple locations. Inter-site connectivity between Headquarters and Branch provides full access to internal networks. Dynamic routing provides stable connectivity, while NAT and PAT allow controlled Internet access from both sites without affecting internal communication. High availability at the Headquarters site is provided by VRRP on core routers, providing redundant default gateways.

Centralized monitoring is implemented using Zabbix to provide visibility of network devices, services, and system health across both sites. Remote administration is demonstrated from a SOHO-based admin workstation using SSH and PuTTY.

The infrastructure includes two internal servers with different roles. One server provides core network services such as DHCP, DNS (dnsmasq), HTTP (Apache2), and centralized logging using rsyslog. The second server is dedicated to monitoring with Zabbix, offering real-time insight into network operations. As a whole, the project demonstrates a practical enterprise design that combines network architecture, core services, centralized monitoring, and real-world operation in a single, consistent solution.


<br>

## **Topology Diagram**


![](images/Pasted%20image%2020260110031623.png)

<br>

## **Network Zones**

- **Internet / ISP**  
    Simulated external Internet connectivity.
    
- **WAN / GRE Overlay**  
    Site-to-site connectivity between Headquarters and Branch.
    
- **Headquarters Routing Zone (HQ-EDGE, HQ-CORE)**  
    Core routing, WAN access, and gateway redundancy.
    
- **Headquarters Distribution Zone (HQ-DIST-SW)**  
    Aggregation between routing and access layers.
    
- **Headquarters Access Zone (HQ-ACC-SW)**  
    End devices and internal servers connectivity.
    
- **Branch Routing Zone (BR1-EDGE)**  
    Local routing and WAN uplink for Branch site.
    
- **Branch Access Zone (BR1-DIST-SW)**  
    Branch end devices connectivity.
    
- **Infrastructure Services**  
    DHCP, DNS, HTTP, centralized logging.
    
- **Monitoring Zone**  
    Centralized monitoring using Zabbix.
    
- **Remote Administration (SOHO)**  
    External SSH access from remote admin workstation.


<br>

## **Project Structure**

1. [Network Topology and Devices](01-network-topology-and-devices.md)
    
2. [Addressing and VLAN Planning](02-addressing-and-vlan-planning.md)
    
3. [Basic Device Configuration](03-basic-device-configuration.md)
    
4. [Switching and VLAN Configuration](04-switching-and-vlan-configuration.md)
    
5. [Inter VLAN Routing and VRRP](05-inter-vlan-routing-and-vrrp.md)
    
6. [GRE Tunnels and Site Connectivity](06-gre-tunnels-and-site-connectivity.md)
    
7. [Core Server Services](07-core-server-services.md)
    
8. [NAT and PAT Configuration](08-nat-pat-configuration.md)
    
9. [Network Access Control and ACL](09-network-access-control-and-acl.md)
    
10. [Centralized Monitoring Zabbix](10-centralized-monitoring-zabbix.md)
    
11. [Troubleshooting](11-troubleshooting.md)
    
12. [Conclusion and Summary](12-conclusion-and-summary.md)


<br>

## **Tools and Environment**

- **GNS3 version 2.2.54**
    
- **Wireshark Version 4.2.2**
    
- **Xubuntu VM** (kernel-based QEMU virtual machine inside GNS3)
    
- **Cisco IOSv Router**
    
    - _VIOS-ADVENTERPRISEK9-M, Version 15.9(3)M6_
        
- **Cisco IOSv-L2 Switch**
    
    - _vios_l2-ADVENTERPRISEK9-M, Version 15.2(20170321)_
        
- **Zabbix Server** (centralized monitoring)
    
- **Visual Studio Code** (documentation editing)
    
- **Obsidian** (notes, summaries and screenshots)


<br>

## **Key Project Features**

- VLAN segmentation and access/trunk port design
    
- Inter-VLAN routing using 802.1Q subinterfaces
    
- Dynamic routing with OSPF (Area 0 and Area 1)
    
- Site-to-site connectivity using a GRE tunnel
    
- Gateway redundancy at Headquarters using VRRP
    
- Layer 2 stability with Rapid Spanning Tree Protocol (RSTP)
    
- Internet access using NAT and PAT on edge routers
    
- Network access control using Access Control Lists (ACL)
    
- Centralized DHCP services provided by Xubuntu server
    
- Internal DNS resolution using dnsmasq
    
- Internal HTTP service using Apache2
    
- Centralized logging using rsyslog
    
- Secure remote administration using SSH and PuTTY
    
- Centralized monitoring and visibility using Zabbix
    
- Real troubleshooting based on issues identified during the project


<br>

## **Author’s Note** 

This enterprise project was the biggest challenge in my learning journey. It was my first time working with two separate parts of a company network, the Headquarters site and the Branch site, and making them operate as one system. Bringing services like DHCP and full connectivity across a GRE tunnel between both sites was challenging, but very satisfying once everything worked correctly.

Centralized monitoring was an important new experience for me. Using Zabbix in an enterprise model and visually observing warnings, outages, and network operation helped me understand the network much better. Centralized logging with rsyslog also added more visibility and control over the infrastructure.

This project brings together everything I learned in previous projects. I am proud of the network diagram, designed purely by logic without using external inspiration. Despite challenges, including the Zabbix installation and real issues described in the troubleshooting chapter, the project was completed successfully. In 2026, this project represents a strong step forward in my understanding of enterprise networking and monitoring.


<br>


---

<br>



© 2026 – Lukáš Dula | Home Network Project & Portfolio



