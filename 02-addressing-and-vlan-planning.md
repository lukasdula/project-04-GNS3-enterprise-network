
<br>

# **2 - Addressing and Vlan Planning**


<br>

## **2.1 Introduction**

This second chapter focuses on IPv4 addressing design and subnet planning for Project 04 Enterprise Network. It provides an overview of how address space is structured across the headquarters and branch site, with clear separation between internal enterprise networks and external connectivity.

Public-looking IPv4 address ranges are used for ISP and Internet-facing links to reflect real-world WAN connectivity, while private address space is used for all internal routing, VLANs, and management traffic. Each functional network segment is assigned its own subnet, including user VLANs, server networks, printer segments, and a dedicated management VLAN 99.

Subnet sizes are selected based on the expected role and scale of each segment. Larger subnets are allocated to user access networks, while smaller prefixes are used for servers, printers, management, and point-to-point routing links. This approach improves clarity, supports simple address utilization, and follows common enterprise network design practices.

<br>

## **2.2  Topology Diagram**

![](images/Pasted%20image%2020260110031133.png)


<br>


## **2.3 Headquarters - IP Addressing and Subnet Overview**

| **Device / Interface**    | **Description**                              | **IP Address**            | **Subnet Mask** | **Default Gateway**   | **Assigned Network** |
| ------------------------- | -------------------------------------------- | ------------------------- | --------------- | --------------------- | -------------------- |
| HQ-EDGE-R3 Gi0/1          | Point-to-point link to HQ-CORE-R1            | 10.50.200.1               | 255.255.255.252 | N/A                   | 10.50.200.0/30       |
| HQ-CORE-R1 Gi0/1          | Point-to-point link to HQ-EDGE-R3            | 10.50.200.2               | 255.255.255.252 | N/A                   | 10.50.200.0/30       |
| HQ-EDGE-R3 Gi0/2          | Point-to-point link to HQ-CORE-R2            | 10.50.200.5               | 255.255.255.252 | N/A                   | 10.50.200.4/30       |
| HQ-CORE-R2 Gi0/2          | Point-to-point link to HQ-EDGE-R3            | 10.50.200.6               | 255.255.255.252 | N/A                   | 10.50.200.4/30       |
| HQ-CORE-R1 Gi0/5          | Inter-core point-to-point link to HQ-CORE-R2 | 10.50.200.9               | 255.255.255.252 | N/A                   | 10.50.200.8/30       |
| HQ-CORE-R2 Gi0/5          | Inter-core point-to-point link to HQ-CORE-R1 | 10.50.200.10              | 255.255.255.252 | N/A                   | 10.50.200.8/30       |
| HQ-CORE-R1 Loopback0      | Core router loopback (router-id)             | 10.50.255.1               | 255.255.255.255 | N/A                   | 10.50.255.1/32       |
| HQ-CORE-R2 Loopback0      | Core router loopback (router-id)             | 10.50.255.2               | 255.255.255.255 | N/A                   | 10.50.255.2/32<br>   |
| HQ-EDGE-R3 Loopback0      | Edge router loopback (router-id)             | 10.50.255.3               | 255.255.255.255 | N/A                   | 10.50.255.3/32       |
| HQ-SRV-INFRA-01           | Infrastructure server (static)               | 10.50.10.10               | 255.255.255.192 | 10.50.10.1 (VRRP VIP) | 10.50.10.0/26        |
| HQ-SRV-MON-01             | Monitoring server (static)                   | 10.50.20.10               | 255.255.255.192 | 10.50.20.1 (VRRP VIP) | 10.50.20.0/26        |
| HQ-OFFICE-01              | Office client (DHCP)                         | 10.50.30.100-10.50.30.199 | 255.255.255.0   | 10.50.30.1 (VRRP VIP) | 10.50.30.0/24        |
| HQ-FINANCE-01             | Finance client (DHCP)                        | 10.50.40.100-10.50.40.199 | 255.255.255.0   | 10.50.40.1 (VRRP VIP) | 10.50.40.0/24        |
| HQ-DIST-SW-01 VLAN 99 SVI | Switch management (static)                   | 10.50.99.10               | 255.255.255.192 | 10.50.99.1 (VRRP VIP) | 10.50.99.0/26        |
| HQ-DIST-SW-02 VLAN 99 SVI | Switch management (static)                   | 10.50.99.20               | 255.255.255.192 | 10.50.99.1 (VRRP VIP) | 10.50.99.0/26        |
| HQ-ACC-SW-01 VLAN 99 SVI  | Switch management (static)                   | 10.50.99.30               | 255.255.255.192 | 10.50.99.1 (VRRP VIP) | 10.50.99.0/26        |
| HQ-ACC-SW-02 VLAN 99 SVI  | Switch management (static)                   | 10.50.99.40               | 255.255.255.192 | 10.50.99.1 (VRRP VIP) | 10.50.99.0/26        |


<br>


>**Notes.:** Each HQ Core Router uses a unique router subinterface IP address within each VLAN subnet (.2 and .3). The VRRP virtual IP (.1) is configured as the default gateway for all end devices.



<br>

## **2.4 VLAN Planning - Headquarters (VRRP)**

| **VLAN ID** | **VLAN Name**  | **Description / Purpose** | **Assigned Network** | **Default Gateway (VRRP VIP)** | **R1 Subinterface IP** | **R2 Subinterface IP** | **Assigned Devices**                                     | **Switch Assoc**             |
| ----------- | -------------- | ------------------------- | -------------------- | ------------------------------ | ---------------------- | ---------------------- | -------------------------------------------------------- | ---------------------------- |
| 10          | Infrastructure | Infrastructure servers    | 10.50.10.0/26        | 10.50.10.1                     | 10.50.10.2             | 10.50.10.3             | HQ-SRV-INFRA-01                                          | HQ-ACC-SW-01                 |
| 20          | Monitoring     | Monitoring servers        | 10.50.20.0/26        | 10.50.20.1                     | 10.50.20.2             | 10.50.20.3             | HQ-SRV-MON-01                                            | HQ-ACC-SW-01                 |
| 30          | Office         | Office user workstations  | 10.50.30.0/24        | 10.50.30.1                     | 10.50.30.2             | 10.50.30.3             | HQ-OFFICE-01                                             | HQ-ACC-SW-02                 |
| 40          | Finance        | Finance user workstations | 10.50.40.0/24        | 10.50.40.1                     | 10.50.40.2             | 10.50.40.3             | HQ-FINANCE-01                                            | HQ-ACC-SW-02                 |
| 99          | Management     | Network device management | 10.50.99.0/26        | 10.50.99.1                     | 10.50.99.2             | 10.50.99.3             | HQ-DIST-SW-01, HQ-DIST-SW-02, HQ-ACC-SW-01, HQ-ACC-SW-02 | HQ-DIST-SW-01, HQ-DIST-SW-02 |

<br>

---

<br>

## **2.5 Branch-01 - IP Addressing and Subnet Overview**

| **Device / Interface**     | **Description**            | **IP Address**            | **Subnet Mask** | **Default Gateway** | **Assigned Network** |
| -------------------------- | -------------------------- | ------------------------- | --------------- | ------------------- | -------------------- |
| BR1-DIST-SW-01 VLAN 99 SVI | Switch management (static) | 10.60.99.50               | 255.255.255.192 | 10.60.99.1          | 10.60.99.0/26        |
| BR1-DIST-SW-02 VLAN 99 SVI | Switch management (static) | 10.60.99.60               | 255.255.255.192 | 10.60.99.1          | 10.60.99.0/26        |
| BR1-OFFICE-01              | Office client (DHCP)       | 10.60.50.100-10.60.50.199 | 255.255.255.0   | 10.60.50.1          | 10.60.50.0/24        |
| BR1-OFFICE-02              | Office client (DHCP)       | 10.60.60.100-10.60.60.199 | 255.255.255.0   | 10.60.60.1          | 10.60.60.0/24        |
| BR1-SALES-01               | Sales client (DHCP)        | 10.60.70.100-10.60.70.199 | 255.255.255.0   | 10.60.70.1          | 10.60.70.0/24        |
| BR1-PRINTER-01             | Network printer (static)   | 10.60.80.10               | 255.255.255.224 | 10.60.80.1          | 10.60.80.0/27        |
| BR1-EDGE-R4-Loopback0      | loopback (router-id)<br>   | 10.60.255.1               | 255.255.255.255 | N/A                 | 10.60.255.1/32       |




<br>

## **2.6 VLAN Planning - Branch-01**

| **VLAN ID** | **VLAN Name** | **Description / Purpose**                   | **Assigned Network** | **Default Gateway** | **Edge Subinterface IP** | **Assigned Devices**           | **Switch Assoc**               |
| ----------- | ------------- | ------------------------------------------- | -------------------- | ------------------- | ------------------------ | ------------------------------ | ------------------------------ |
| 50          | Office-01     | Branch office user workstations (Office-01) | 10.60.50.0/24        | 10.60.50.1          | 10.60.50.1               | BR1-OFFICE-01                  | BR1-DIST-SW-01                 |
| 60          | Office-02     | Branch office user workstations (Office-02) | 10.60.60.0/24        | 10.60.60.1          | 10.60.60.1               | BR1-OFFICE-02                  | BR1-DIST-SW-02                 |
| 70          | Sales         | Branch sales workstations                   | 10.60.70.0/24        | 10.60.70.1          | 10.60.70.1               | BR1-SALES-01                   | BR1-DIST-SW-01                 |
| 80          | Printer       | Branch network printer segment              | 10.60.80.0/27        | 10.60.80.1          | 10.60.80.1               | BR1-PRINTER-01                 | BR1-DIST-SW-02                 |
| 99          | Management    | Branch network device management            | 10.60.99.0/26        | 10.60.99.1          | 10.60.99.1               | BR1-DIST-SW-01, BR1-DIST-SW-02 | BR1-DIST-SW-01, BR1-DIST-SW-02 |



<br>

---


<br>

## **2.7 WAN, ISP, Internet-Cloud, Remote-Admin, and GRE Subnet Overview**

| **Device / Interface**  | **Description**                               | **IP Address** | **Subnet Mask** | **Default Gateway** | **Assigned Network** |
| ----------------------- | --------------------------------------------- | -------------- | --------------- | ------------------- | -------------------- |
| HQ-ISP Gi0/5            | ISP to HQ-EDGE-R3 point-to-point link         | 37.48.0.1      | 255.255.255.252 | N/A                 | 37.48.0.0/30         |
| HQ-EDGE-R3 Gi0/5        | HQ-EDGE-R3 to HQ-ISP point-to-point link      | 37.48.0.2      | 255.255.255.252 | N/A                 | 37.48.0.0/30         |
| BR1-ISP Gi0/5           | ISP to BR1-EDGE-R4 point-to-point link        | 89.203.0.1     | 255.255.255.252 | N/A                 | 89.203.0.0/30        |
| BR1-EDGE-R4 Gi0/5       | BR1-EDGE-R4 to BR1-ISP point-to-point link    | 89.203.0.2     | 255.255.255.252 | N/A                 | 89.203.0.0/30        |
| HQ-ISP Gi0/0            | HQ-ISP uplink to Internet-Cloud               | 46.23.10.1     | 255.255.255.252 | N/A                 | 46.23.10.0/30        |
| INTERNET-CLOUD Gi0/0    | Internet-Cloud to HQ-ISP uplink               | 46.23.10.2     | 255.255.255.252 | N/A                 | 46.23.10.0/30        |
| BR1-ISP Gi0/1           | BR1-ISP uplink to Internet-Cloud              | 93.184.216.1   | 255.255.255.252 | N/A                 | 93.184.216.0/30      |
| INTERNET-CLOUD<br>Gi0/1 | Internet-Cloud to BR1-ISP uplink              | 93.184.216.2   | 255.255.255.252 | N/A                 | 93.184.216.0/30      |
| REMOTE-ADMIN Gi0/0      | Remote administrator client (SOHO)            | 185.10.10.2    | 255.255.255.252 | 185.10.10.1         | 185.10.10.0/30       |
| INTERNET-CLOUD Gi0/2    | Internet-Cloud to Remote-Admin (SOHO) segment | 185.10.10.1    | 255.255.255.252 | N/A                 | 185.10.10.0/30       |
| HQ-EDGE-R3 Tunnel0      | GRE overlay tunnel interface (HQ side)        | 10.254.0.1     | 255.255.255.252 | N/A                 | 10.254.0.0/30        |
| BR1-EDGE-R4 Tunnel0     | GRE overlay tunnel interface (Branch side)    | 10.254.0.2     | 255.255.255.252 | N/A                 | 10.254.0.0/30        |



<br>


## **2.8 Conclusion**

This chapter defines a clear and structured IPv4 addressing and VLAN planning model for the enterprise network. Address space is consistently divided between headquarters, branch, and external connectivity, with each functional segment placed in its own subnet. Internal networks use private addressing, while ISP and Internet-facing links use public-looking address ranges to reflect realistic WAN design.

The resulting addressing scheme supports scalability, readability, and straightforward routing configuration. VLAN assignments, subnet sizes, and gateway placement follow common enterprise design principles, providing a solid foundation for redundancy, dynamic routing, GRE tunneling, and security features configured in later chapters.

<br>

---


<br>

**Next Chapter:**