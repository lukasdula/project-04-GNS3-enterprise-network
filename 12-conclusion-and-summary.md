
<br>

# 12 - Conclusion and Summary


<br>

## Final Evaluation

This enterprise project defines a complete multi-site network that operates as an integrated routed and switched infrastructure. The design focuses on availability, segmentation, centralized services, a basic security baseline, and operational verification across Headquarters and Branch locations. Core technologies are combined into a single system, including VLAN segmentation, inter-VLAN routing, dynamic routing with OSPF, GRE tunnel connectivity, NAT/PAT for Internet access, centralized monitoring, and access control.

The topology follows a layered enterprise design with core, distribution, and access roles. Headquarters and Branch sites are interconnected through a GRE tunnel over a simulated ISP, enabling consistent inter-site communication. Remote administrative access is verified using PuTTY over SSH through the management network, confirming secure device management across sites. IPv4 addressing and VLAN planning clearly separate infrastructure, management, monitoring, servers, and user networks, with VLAN 99 reserved for centralized management.

Layer 2 and Layer 3 connectivity are implemented using VLANs, trunk links, and 802.1Q subinterfaces. OSPF provides dynamic routing and stable reachability across both sites, while VRRP delivers default gateway redundancy for user VLANs in the headquarters network. Static and default routes are applied at the network edge where required. NAT and PAT translate internal private addresses to public WAN addresses, enabling controlled Internet access without impacting internal routing and services.

Core services are centralized on internal servers and include DHCP, DNS, HTTP, and Rsyslog logging. Centralized monitoring with Zabbix provides visibility into device availability and overall network health. Access Control Lists apply a simple but effective security policy that allows required services while limiting unnecessary inter-VLAN communication.

Operational testing and troubleshooting are based on real issues that appeared during the network build. Routing issues, NAT/PAT operation, and missing return paths in the management network are identified, fixed, and verified. After troubleshooting, the network operates as designed, with stable inter-site connectivity, functional services, applied access control, and reliable monitoring. The project presents a realistic enterprise network model that connects design, implementation, verification, and troubleshooting into one complete solution.



<br>

## **Topology Diagram**


![](images/Pasted%20image%2020260119234715.png)


<br>

## **Project Overview**

1. [Network Topology and Devices](01-network-topology-and-devices.md)
    The project begins with the definition of an enterprise topology for Headquarters and Branch sites. Core, distribution, and access roles are assigned to routers and switches within a layered design that establishes a clear physical and logical foundation for scalability, redundancy, and role separation.
    
2. [Addressing and VLAN Planning](02-addressing-and-vlan-planning.md) 
    IPv4 addressing and VLAN segmentation separate infrastructure, management, monitoring, servers, and user networks. Subnet sizes reflect network purpose and expected host counts. VLAN 99 is reserved for centralized management across both sites.
    
3. [Basic Device Configuration](03-basic-device-configuration.md)
    Baseline configuration is applied to all routers and switches, including hostnames, interface addressing, port descriptions, and basic administrative settings. This chapter provides a consistent and stable starting point for further configuration.
    
4. [Switching and VLAN Configuration](04-switching-and-vlan-configuration.md) 
    VLANs are configured on all switches, with access and trunk ports assigned according to the design. Rapid Spanning Tree Protocol, PortFast, BPDU Guard, and link aggregation are applied to ensure stable Layer 2 connectivity and protect the switching environment.
    
5. [Inter VLAN Routing and VRRP](05-inter-vlan-routing-and-vrrp.md)
    Inter-VLAN routing is implemented using 802.1Q subinterfaces on the headquarters core routers. OSPF supports dynamic routing within the site, while VRRP provides default gateway redundancy for user VLANs, ensuring continuous access during gateway failures.
    
6. [GRE Tunnels and Site Connectivity](06-gre-tunnels-and-site-connectivity.md) 
    Headquarters and Branch networks are interconnected through a GRE tunnel over a simulated ISP. OSPF exchanges routing information across the tunnel, enabling reliable inter-site connectivity and shared services between locations.
    
7. [Core Server Services](07-core-server-services.md) 
    Centralized infrastructure services are deployed on internal servers. DHCP assigns addresses using relay, DNS services are provided by dnsmasq, HTTP access runs on Apache2, and logging Rsyslog supports monitoring and troubleshooting activities.
    
8. [NAT and PAT Configuration](08-nat-pat-configuration.md) 
    NAT and PAT are configured on both headquarters and branch edge routers to provide controlled Internet access for internal networks. Address translation is verified without disrupting internal routing or inter-site communication.
    
9. [Network Access Control and ACL](09-network-access-control-and-acl.md)
    Remote administrative access is demonstrated using PuTTY and SSH from a Remote-Admin (SOHO) workstation into the enterprise network. Secure access to branch routers and switches is verified before Access Control Lists apply a basic security baseline across both sites.
    
10. [Centralized Monitoring Zabbix](10-centralized-monitoring-zabbix.md)
    Centralized monitoring is implemented using Zabbix. Five monitoring scenarios are validated, covering device availability, interface status, ICMP reachability, service monitoring, and inter-site connectivity, ensuring continuous visibility into network health.
    
11. [Troubleshooting](11-troubleshooting.md)
    This chapter documents four real issues that appeared during the network build. Problems related to routing, NAT/PAT operation, and management return paths are identified, fixed, and verified, demonstrating a structured troubleshooting approach.
    


<br>

---




<br>


**Back to project overview:** [README](README.md)
















