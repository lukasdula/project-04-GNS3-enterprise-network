
<br>

# **1 - Network Topology and Devices**

<br>

## **1.1  Introduction**

This first chapter introduces the enterprise network topology used in Project 04. The design represents a headquarters site and a branch office connected through a simulated Internet service provider using a GRE tunnel. A remote administrator connects from a SOHO environment and accesses the network through the Internet.

The network follows a layered architecture that separates edge routing, core routing, distribution switching, and access switching. Headquarters uses redundant routing components and a resilient switching design to support availability and fault tolerance, while the branch site applies a simplified architecture based on the same design principles.

Wide area connectivity is provided by edge routers at each location. Inter-site communication is carried through a GRE tunnel built over the simulated Internet. Static and dynamic routing mechanisms are used to establish controlled connectivity between internal networks and external services.

This chapter describes the topology diagram, network zones, device roles, and connection structure that support routing, resiliency, and infrastructure services configured in later chapters.

<br>

## **1.2  Topology Diagram**

![](images/Pasted%20image%2020260110030214.png)

<br>

## **1.3  Network Zones**

This section describes the main parts of the topology and their roles.

- **Internet and ISP**
    
    - Simulated external connectivity to the Internet.
        
- **WAN and GRE Overlay**
    
    - Logical site-to-site connectivity between headquarters and branch.
        
    - GRE tunnel built over the simulated Internet.
        
- **Headquarters Routing Zone (HQ-EDGE, HQ-CORE)**
    
    - Provides Layer 3 connectivity for internal VLANs.
        
    - Handles upstream connectivity to the ISP.
        
    - Uses gateway and routing redundancy.
        
- **Headquarters Distribution Zone (HQ-DIST-SW)**
    
    - Connects core routing and access layers.
        
- **Headquarters Access Zone (HQ-ACC-SW)**
    
    - Connects end devices and servers to the network.
        
- **Branch Routing Zone (BR1-EDGE)**
    
    - Provides local routing at the branch site.
        
    - Connects the branch network to the WAN.
        
- **Branch Access Zone (BR1-DIST-SW)**
    
    - Connects branch end devices to the network.
        
- **Infrastructure Services**
    
        
    - Includes DHCP, DNS, and a local HTTP service for internal use.
        
- **Monitoring Server**
    
    - Centralized network monitoring using Zabbix.
        
- **Remote Administration (SOHO)**
    
    - Remote administrator connecting from a home network over the Internet.

<br>


## **1.4 Used Devices**

| **Device**                     | **Image / Type** | **Interfaces**     | **Access** | **Purpose**                                                                                              |
| ------------------------------ | ---------------- | ------------------ | ---------- | -------------------------------------------------------------------------------------------------------- |
| HQ-CORE-R1                     | Cisco IOSv       | 6x GigabitEthernet | Console    | Primary core routing device; inter-VLAN routing; VRRP master gateway; uplink to HQ edge router.          |
| HQ-CORE-R2                     | Cisco IOSv       | 6x GigabitEthernet | Console    | Secondary core routing device; inter-VLAN routing backup; VRRP backup gateway; uplink to HQ edge router. |
| HQ-EDGE-R3                     | Cisco IOSv       | 6x GigabitEthernet | Console    | Edge router providing WAN connectivity to ISP and GRE tunnel endpoint for headquarters.                  |
| BR1-EDGE-R4                    | Cisco IOSv       | 6x GigabitEthernet | Console    | Branch edge router providing WAN connectivity to ISP and GRE tunnel endpoint for branch site.            |
| HQ-DIST-SW-01, HQ-DIST-SW-02   | Cisco IOSv-L2    | 4x GigabitEthernet | Console    | Distribution switches; 802.1Q VLAN trunking                                                              |
| HQ-ACC-SW-01, HQ-ACC-SW-02     | Cisco IOSv-L2    | 4x GigabitEthernet | Console    | Access switches connecting end devices and servers at headquarters.                                      |
| BR1-DIST-SW-01, BR1-DIST-SW-02 | Cisco IOSv-L2    | 4x GigabitEthernet | Console    | Branch switches providing access connectivity for branch end devices.                                    |
| HQ-SRV-INFRA-01                | QEMU Xubuntu VM  | 1x GigabitEthernet | VNC        | Infrastructure server providing DHCP, DNS, and local HTTP services.                                      |
| HQ-SRV-MON-01                  | QEMU Xubuntu VM  | 1x GigabitEthernet | VNC        | Monitoring server running Zabbix for centralized network monitoring.                                     |
| Remote-Admin                   | QEMU Xubuntu VM  | 1x GigabitEthernet | VNC        | Remote administration workstation connecting from a SOHO environment over the Internet.                  |
| HQ-OFFICE-01                   | VPCS             | 1x Ethernet        | Console    | Office client device connected to the headquarters access network.                                       |
| HQ-FINANCE-01                  | VPCS             | 1x Ethernet        | Console    | Finance client device connected to the headquarters access network.                                      |
| BR1-OFFICE-01                  | VPCS             | 1x Ethernet        | Console    | Office client device connected to the branch access network.                                             |
| BR1-SALES-01                   | VPCS             | 1x Ethernet        | Console    | Sales client device connected to the branch access network.                                              |
| BR1-PRINTER-01                 | VPCS             | 1x Ethernet        | Console    | Network printer connected to the branch access network.                                                  |
| Internet-Cloud                 | Cisco IOSv       | 6x GigabitEthernet | Console    | Simulated Internet cloud representing external networks and transit between ISP routers.                 |
| BR1-ISP                        | Cisco IOSv       | 6x GigabitEthernet | Console    | ISP router providing upstream Internet connectivity.                                                     |
| HQ-ISP                         | Cisco IOSv       | 6x GigabitEthernet | Console    | ISP edge router providing upstream Internet connectivity for headquarters                                |




<br>


## **1.5 Connection Overview – Headquarters**

| **Device**    | **Interface** | **Connected to** | **Peer Interface** | **Purpose**                                                                     |
| ------------- | ------------- | ---------------- | ------------------ | ------------------------------------------------------------------------------- |
| HQ-ISP        | Gi0/5         | HQ-EDGE-R3       | Gi0/5              | WAN uplink between ISP router and headquarters edge router.                     |
| HQ-EDGE-R3    | Gi0/1         | HQ-CORE-R1       | Gi0/1              | Uplink from edge router to primary core router.                                 |
| HQ-EDGE-R3    | Gi0/2         | HQ-CORE-R2       | Gi0/2              | Uplink from edge router to secondary core router.                               |
| HQ-CORE-R1    | Gi0/5         | HQ-CORE-R2       | Gi0/5              | Core router interconnection for redundancy and routing synchronization.         |
| HQ-CORE-R1    | Gi0/0         | HQ-DIST-SW-01    | Gi0/0              | Downlink from core router to distribution switch.                               |
| HQ-CORE-R2    | Gi0/0         | HQ-DIST-SW-02    | Gi0/0              | Downlink from core router to distribution switch.                               |
| HQ-DIST-SW-01 | Gi0/3         | HQ-DIST-SW-02    | Gi0/1              | Inter-switch link providing Layer 2 connectivity between distribution switches. |
| HQ-DIST-SW-01 | Gi0/2         | HQ-ACC-SW-01     | Gi0/1              | Downlink from distribution switch to access switch.                             |
| HQ-DIST-SW-02 | Gi0/3         | HQ-ACC-SW-02     | Gi0/2              | Downlink from distribution switch to access switch.                             |
| HQ-ACC-SW-01  | Gi0/0         | HQ-SRV-INFRA-01  | Gi0/0              | Access link for infrastructure server VLAN.                                     |
| HQ-ACC-SW-01  | Gi0/2         | HQ-SRV-MON-01    | Gi0/0              | Access link for monitoring server VLAN.                                         |
| HQ-ACC-SW-02  | Gi0/0         | HQ-OFFICE-01     | e0                 | Access link for office client VLAN.                                             |
| HQ-ACC-SW-02  | Gi0/1         | HQ-FINANCE-01    | e0                 | Access link for finance client VLAN.                                            |
| HQ-ACC-SW-01  | Gi0/3         | HQ-ACC-SW-02     | Gi0/3              | Inter-switch link providing Layer 2 connectivity between distribution switches. |

<br>


## **1.6 Connection Overview – Branch-01**

| **Device**     | **Interface** | **Connected to** | **Peer Interface** | **Purpose**                                                                     |
| -------------- | ------------- | ---------------- | ------------------ | ------------------------------------------------------------------------------- |
| BR1-ISP        | Gi0/5         | BR1-EDGE-R4      | Gi0/5              | WAN uplink between ISP router and branch edge router.                           |
| BR1-EDGE-R4    | Gi0/1         | BR1-DIST-SW-01   | Gi0/1              | Uplink from branch edge router to distribution switch.                          |
| BR1-EDGE-R4    | Gi0/2         | BR1-DIST-SW-02   | Gi0/2              | Uplink from branch edge router to distribution switch.                          |
| BR1-DIST-SW-01 | Gi0/0         | BR1-DIST-SW-02   | Gi0/0              | Inter-switch link providing Layer 2 connectivity between distribution switches. |
| BR1-DIST-SW-01 | Gi0/2         | BR1-OFFICE-01    | e0                 | Access link for branch office client.                                           |
| BR1-DIST-SW-01 | Gi0/3         | BR1-SALES-01     | e0                 | Access link for branch sales client.                                            |
| BR1-DIST-SW-02 | Gi0/1         | BR1-OFFICE-02    | e0                 | Access link for branch office client.                                           |
| BR1-DIST-SW-02 | Gi0/3         | BR1-PRINTER-01   | e0                 | Access link for branch network printer.                                         |


<br>


## **1.7 WAN and GRE Tunnel Connections**

| **Device**   | **Interface** | **Connected to** | **Peer Interface** | **Purpose**                                                                                      |
| ------------ | ------------- | ---------------- | ------------------ | ------------------------------------------------------------------------------------------------ |
| HQ-ISP       | Gi0/0         | Internet-Cloud   | Gi0/0              | WAN connection between headquarters ISP router and the Internet.                                 |
| BR1-ISP      | Gi0/1         | Internet-Cloud   | Gi0/1              | WAN connection between branch ISP router and the Internet.                                       |
| HQ-ISP       | Gi0/5         | HQ-EDGE-R3       | Gi0/5              | WAN uplink between ISP router and headquarters edge router.                                      |
| BR1-ISP      | Gi0/5         | BR1-EDGE-R4      | Gi0/5              | WAN uplink between ISP router and branch edge router.                                            |
| HQ-EDGE-R3   | Tunnel0       | BR1-EDGE-R4      | Tunnel0            | GRE tunnel providing site-to-site connectivity between headquarters and branch.                  |
| Remote-Admin | Gi0/0         | Internet-Cloud   | N/A                | Remote administrator accessing the enterprise network from a SOHO environment over the Internet. |


<br>

## **1.8 Conclusion**

This chapter defines the enterprise network topology used in Project 04 and describes its main structural components. The design separates headquarters and branch sites into clear routing, distribution, and access layers, while WAN connectivity is provided through a simulated Internet service and a GRE site-to-site tunnel. Device roles, network zones, and physical connections are documented to establish a clear overview of the network architecture.

The presented topology provides a solid foundation for further configuration of addressing, VLAN segmentation, routing, redundancy, and infrastructure services. Subsequent chapters build on this structure by defining IP addressing, VLAN assignments, and detailed device configurations required for operational network functionality.

<br>

---


<br>

**Next Chapter:** [Addressing and VLAN Planning](02-addressing-and-vlan-planning.md)
