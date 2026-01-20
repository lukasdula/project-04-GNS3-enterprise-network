
<br>

# **3 - Basic Device Configuration**

<br>

## **3.1 Introduction**

This first configuration section establishes a clean and stable baseline for the enterprise network before any Layer 2, routing, or security features are introduced. The purpose of this chapter is to prepare all core devices with basic configuration so the topology is reachable and ready for next configuration steps in later chapters.

Static IPv4 addressing is applied only to infrastructure servers, which require fixed and predictable network addresses for management and monitoring. End-user devices are intentionally left unconfigured at this stage and will be addressed in later chapters.

All routers and switches receive consistent hostnames, and IPv4 addresses are configured on all WAN and inter-router point-to-point links. Core and edge routers are also assigned loopback interfaces, which provide stable logical addresses used for router identification and internal routing purposes with monitoring for zabbix. Interface descriptions are added to make link roles easier to understand, and the chapter concludes with basic connectivity verification using interface checks and ICMP tests.

<br>

## **3.2 Topology Diagram**

![](images/Pasted%20image%2020260111034919.png)

<br>

## **3.3 Objectives**

1. Assign static IPv4 addresses to infrastructure servers and selected network devices, including management, monitoring, and printer segments.
    
2. Configure hostnames on all routers and switches to ensure consistent device identification across the topology.
    
3. Configure IPv4 addressing on all WAN and inter-router point-to-point links between headquarters, branch, the ISP, and the Internet, including loopback interfaces on core/edge routers.
    
4. Apply interface descriptions on routers and switches to clearly show link purpose and connected devices.
    
5. Verify basic Layer 3 connectivity and router-to-router communication using interface status checks and ICMP tests.


<br>

## **3.4 Static IPv4 Addressing - Infrastructure Servers**


Static IPv4 addressing is configured on enterprise infrastructure servers to ensure predictable and stable network connectivity. These hosts provide core services for the internal network and must stay reachable at fixed IP addresses across all configuration stages.

Address configuration is applied using Netplan so that settings stay active after system reboot. Only infrastructure servers are configured in this section; end-user devices and remote administration hosts are not configured at this stage.

<br>

### **Addressing Overview**

| **Device**      | **Role**                                  | **IPv4 Address** | **Subnet Mask** | **Default Gateway**   | **Network**   |
| --------------- | ----------------------------------------- | ---------------- | --------------- | --------------------- | ------------- |
| HQ-SRV-INFRA-01 | Infrastructure services (DHCP, DNS, HTTP) | 10.50.10.10      | 255.255.255.192 | 10.50.10.1 (VRRP VIP) | 10.50.10.0/26 |
| HQ-SRV-MON-01   | Monitoring server (Zabbix)                | 10.50.20.10      | 255.255.255.192 | 10.50.20.1 (VRRP VIP) | 10.50.20.0/26 |

<br>

### **Static IPv4 Configuration - HQ-SRV-INFRA-01**

Open configuration file:

```plaintext
sudo nano /etc/netplan/01-network-manager-all.yaml
```
![](images/Pasted%20image%2020260108233607.png)

Configuration:

```plaintext
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens3:
      addresses: [10.50.10.10/26]
      nameservers:
        addresses: [10.50.10.1]
      routes:
        - to: 0.0.0.0/0
          via: 10.50.10.1
```
![](images/Pasted%20image%2020260108233955.png)


Apply configuration:

```plaintext
sudo netplan apply
```
![](images/Pasted%20image%2020260108234037.png)

Verification:

```plaintext
ip a
```
![](images/Pasted%20image%2020260108234024.png)


<br>

### **Static IPv4 Configuration - HQ-SRV-MON-01**

Open configuration file:

```plaintext
sudo nano /etc/netplan/01-network-manager-all.yaml
```
![](images/Pasted%20image%2020260108234916.png)

Configuration:

```plaintext
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens3:
      addresses: [10.50.20.10/26]
      nameservers:
        addresses: [10.50.20.1]
      routes:
        - to: 0.0.0.0/0
          via: 10.50.20.1
```
![](images/Pasted%20image%2020260108234859.png)

Apply configuration:

```plaintext
sudo netplan apply
```


Verification:

```plaintext
ip a
```
![](images/Pasted%20image%2020260108234943.png)

<br>

### **Results**

- Both infrastructure servers are configured with fixed static IPv4 addressing using Netplan
    
- Default gateways point to the VRRP virtual IP addresses, providing a redundant gateway path.
    
- Interface configuration is verified locally on each server.
    


<br>

### **Printer Static Configuration**


This part configures a static IPv4 address for the branch network printer. The printer is implemented as a VPCS device, which uses a simplified command set for IP configuration. DHCP is not used for this device to ensure a fixed and predictable address within the printer VLAN.

#### **Printer IP Configuration (VPCS)**

Device:

- **BR1-PRINTER-01**
    

>Note: VPCS does not support full hostname configuration. The printer is identified by the GNS3 node name, while VPCS internal naming is limited and not used for documentation purposes.

**Network parameters:**

- VLAN: 80 (Printer)
    
- Subnet: 10.60.80.0/27
    
- IP address: 10.60.80.10
    
- Default gateway: 10.60.80.1
    

**Configure static IP address:**

```prikaz
ip 10.60.80.10/27 10.60.80.1
```
![](images/Pasted%20image%2020260112235744.png)


**Save configuration:**

```prikaz
save
```
![](images/Pasted%20image%2020260112235752.png)


### **Verification**

Verify IP configuration on the printer:

```prikaz
show ip
```
![](images/Pasted%20image%2020260112235811.png)


### Results

- The printer is assigned a static IPv4 address in the printer VLAN.
    



### **Conclusion**

Static IPv4 addressing is configured for enterprise infrastructure servers and the network printer. These statically addressed endpoints ensure predictable connectivity for internal services and monitoring, and establish a stable foundation for subsequent routing and switching configuration.


<br>

## **3.5 Device Hostname Configuration**

This next section defines hostname configuration for all routing and switching devices in the enterprise network. Hostnames are assigned in a simple and consistent format so that each device is easy to identify during configuration, verification, troubleshooting, and later expansion of the topology.

Only routers and switches are configured in this step. End devices and servers already use predefined names assigned during topology creation and do not require additional hostname configuration at this stage.



### **Hostname Overview – Headquarters**

| **Device Role**         | **Device Name** | **Hostname**  |
| ----------------------- | --------------- | ------------- |
| Core Router (Primary)   | HQ-CORE-R1      | HQ-CORE-R1    |
| Core Router (Secondary) | HQ-CORE-R2      | HQ-CORE-R2    |
| Edge Router             | HQ-EDGE-R3      | HQ-EDGE-R3    |
| Distribution Switch     | HQ-DIST-SW-01   | HQ-DIST-SW-01 |
| Distribution Switch     | HQ-DIST-SW-02   | HQ-DIST-SW-02 |
| Access Switch           | HQ-ACC-SW-01    | HQ-ACC-SW-01  |
| Access Switch           | HQ-ACC-SW-02    | HQ-ACC-SW-02  |

<br>

### **Hostname Overview – Branch-01**

| **Device Role**     | **Device Name** | **Hostname**   |
| ------------------- | --------------- | -------------- |
| Edge Router         | BR1-EDGE-R4     | BR1-EDGE-R4    |
| Distribution Switch | BR1-DIST-SW-01  | BR1-DIST-SW-01 |
| Distribution Switch | BR1-DIST-SW-02  | BR1-DIST-SW-02 |

<br>

### **Hostname Overview – WAN / ISP / Internet**

| **Device Role**     | **Device Name** | **Hostname**   |
| ------------------- | --------------- | -------------- |
| ISP Router (HQ)     | HQ-ISP          | HQ-ISP         |
| ISP Router (Branch) | BR1-ISP         | BR1-ISP        |
| Internet Cloud      | Internet-Cloud  | INTERNET-CLOUD |

<br>

### **Hostname Configuration Example – HQ-EDGE-R3**

The example below demonstrates hostname configuration on the headquarters edge router. All other routers and switches are configured in the same way, with only the hostname value changed based on the table.

Configuration:

```plaintext
enable
configure terminal
hostname HQ-EDGE-R3
exit
write memory
```
![](images/Pasted%20image%2020260108044104.png)

<br>

### **Results**

All routers and switches now use clear and consistent hostnames that reflect their role and location in the enterprise topology. Devices are easily identifiable in the CLI, diagnostic output, and documentation, providing a clean baseline for all subsequent configuration stages.


<br>

## **3.6 IPv4 Configuration - WAN, ISP, and Edge Routers**


This section assigns static IPv4 addresses to router interfaces that form the WAN and Internet-facing part of the enterprise topology. These interfaces connect the enterprise network to the simulated Internet service provider and provide the Layer 3 foundation for external and inter-site connectivity.

Only router interfaces participating in WAN and point-to-point connections are configured in this step. No static routes, default routes, or routing protocols are configured at this stage. The goal is to establish correct interface addressing and link readiness before any routing logic is introduced in later chapters.



### **Scope and Structure**

To keep the configuration clear and easy to follow, this chapter is divided logically:

- ISP and Internet-Cloud routers
    
- Headquarters edge router
    
- Branch edge router
    

Core routers and internal interfaces are configured in the next subsection.

<br>
<br>

### **ISP Router Configuration - HQ-ISP**

Links:

    
- Gi0/5 -> HQ-EDGE-R3 Gi0/5 (37.48.0.0/30)
    
- Gi0/0 -> Internet-Cloud Gi0/0 (46.23.10.0/30)
    

Configuration:

```plaintext
enable
configure terminal
interface GigabitEthernet0/5
ip address 37.48.0.1 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/0
ip address 46.23.10.1 255.255.255.252
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260108044553.png)

### **Ping Tests**

Ping tests verify basic reachability between directly connected devices.

    
- **HQ-ISP → HQ-EDGE-R3** (37.48.0.0/30)
    

```plaintext
ping 37.48.0.1
```
![](images/Pasted%20image%2020260108045054.png)
<br>

### **ISP Router Configuration - BR1-ISP**

Links:

    
- Gi0/5 -> BR1-EDGE-R4 Gi0/5 (89.203.0.0/30)
    
- Gi0/1 -> Internet-Cloud Gi0/1 (93.184.216.0/30)
    

Configuration:

```plaintext
enable
configure terminal
interface GigabitEthernet0/5
ip address 89.203.0.1 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/1
ip address 93.184.216.1 255.255.255.252
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260108050318.png)

<br>

### **Internet-Cloud Router Configuration**

Links:

    
* Gi0/0 -> HQ-ISP Gi0/0 (46.23.10.0/30)
    
- Gi0/1 -> BR1-ISP Gi0/1 (93.184.216.0/30)
    
* Gi0/2 -> REMOTE-ADMIN (SOHO-185.10.10.0/30)
    

Configuration:

```plaintext
enable
configure terminal
interface GigabitEthernet0/0
ip address 46.23.10.2 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/1
ip address 93.184.216.2 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/2
ip address 185.10.10.1 255.255.255.252
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260108045508.png)

<br>

### **Edge Router Configuration - Headquarters (HQ-EDGE-R3)**

Links:

    
- Gi0/5 -> HQ-ISP Gi0/5 (37.48.0.0/30)
    
- Gi0/1 -> HQ-CORE-R1 Gi0/1 (10.50.200.0/30)
    
- Gi0/2 -> HQ-CORE-R2 Gi0/2 (10.50.200.4/30)
    

Configuration:

```plaintext
enable
configure terminal
interface GigabitEthernet0/5
ip address 37.48.0.2 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/1
ip address 10.50.200.1 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/2
ip address 10.50.200.5 255.255.255.252
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260108044139.png)

<br>

### **Edge Router Configuration - Branch (BR1-EDGE-R4)**

Links:

    
- Gi0/5 -> BR1-ISP Gi0/5 (89.203.0.0/30)
    

Configuration:

```plaintext
enable
configure terminal
interface GigabitEthernet0/5
ip address 89.203.0.2 255.255.255.252
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260108050603.png)

<br>

### **Verification**

Example verification on HQ-EDGE-R3:

````plaintext
show ip interface brief
````
![](images/Pasted%20image%2020260108050825.png)

<br>

### **Results**

All WAN, ISP, and edge router interfaces are configured with correct static IPv4 addresses and are in an operational state. Point-to-point links are ready for routing configuration and connectivity testing in subsequent sections.



<br>

## **3.7 IPv4 Configuration - Headquarters Core Routers**

This next configuration section assigns static IPv4 addresses to headquarters core router interfaces used for point-to-point connectivity. The goal is to bring core interconnections up with correct addressing before any VLAN subinterfaces, routing protocols, or routing policies are configured.

Only Layer 3 point-to-point links and loopback interfaces are configured in this step. Loopback interfaces provide a stable logical address used for router identification and internal routing functions. Interfaces used for Router-on-a-Stick towards distribution switches are intentionally not addressed on the physical interface and are configured later using 802.1Q subinterfaces.



### **Core Router Configuration - HQ-CORE-R1**

Links:

    
- Gi0/1 -> HQ-EDGE-R3 Gi0/1 (10.50.200.0/30)
    
- Gi0/5 -> HQ-CORE-R2 Gi0/5 (10.50.200.8/30)
    
- Loopback0 -> Router ID (10.50.255.1/32)
    

Configuration:

```plaintext
enable
configure terminal
interface GigabitEthernet0/1
ip address 10.50.200.2 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/5
ip address 10.50.200.9 255.255.255.252
no shutdown
exit
interface Loopback0
ip address 10.50.255.1 255.255.255.255
exit
end
write memory
```
![](images/Pasted%20image%2020260108042046.png)


<br>

### **Core Router Configuration - HQ-CORE-R2**

Links:

    
- Gi0/2 -> HQ-EDGE-R3 Gi0/2 (10.50.200.4/30)
    
- Gi0/5 -> HQ-CORE-R1 Gi0/5 (10.50.200.8/30)
    
- Loopback0 -> Router ID (10.50.255.2/32)
    

Configuration:

```plaintext
enable
configure terminal
interface GigabitEthernet0/2
ip address 10.50.200.6 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/5
ip address 10.50.200.10 255.255.255.252
no shutdown
exit
interface Loopback0
ip address 10.50.255.2 255.255.255.255
exit
end
write memory
```
![](images/Pasted%20image%2020260108042128.png)

<br>

## Headquarters - HQ-EDGE-R3 Loopback Interface

On the headquarters edge router (HQ-EDGE-R3), a loopback interface is configured to provide a stable router identifier and a reliable management endpoint that does not depend on the state of physical interfaces.

## Configuration

```plaintext
enable
configure terminal
interface Loopback0
description Loopback-HQ-EDGE-R3
ip address 10.50.255.3 255.255.255.255
no shutdown
end
write
```
![](images/Pasted%20image%2020260111033315.png)


<br>


### **Loopback Monitoring (Inter-Core)**

Loopback interfaces configured on core routers HQ-CORE-R1 and HQ-CORE-R2 provide stable logical endpoints for monitoring and verification. Using Wireshark, traffic is observed on the inter-core link between HQ-CORE-R1 and HQ-CORE-R2 to verify that loopback communication is correctly established and routed between the core devices.

![](images/Pasted%20image%2020260110032433.png)

<br>

#### **Loopback Traffic Identification (Wireshark Filter)**

Loopback traffic is identified in Wireshark by applying a display filter that matches the loopback IP addresses of the core routers. This allows focused inspection of packets exchanged between the loopback interfaces.

```plaintext
loop
```
![](images/Pasted%20image%2020260108042619.png)


**Results**

The packet capture confirms successful communication between the loopback interfaces on HQ-CORE-R1 and HQ-CORE-R2. The output demonstrates that inter-core routing is operational and that loopback reachability between the core routers is functioning as expected.

<br>

### **Verification**

Example verification on **HQ-CORE-R1**:

```plaintext
show ip interface brief
```
![](images/Pasted%20image%2020260108043811.png)

<br>

### Results

Headquarters core router point-to-point interfaces and loopback interfaces are configured with static IPv4 addressing. Core-to-edge and core-to-core links are operational and ready for routing and VLAN subinterface configuration in subsequent sections.


<br>

### **Branch-01 - BR1-EDGE-R4 Loopback Interface**


On the Branch-01 edge router (BR1-EDGE-R4), a loopback interface is configured primarily for stable remote administrative access. In enterprise networks, loopbacks provide a permanent and reliable management endpoint independent of physical interface state.

### Configuration

```plaintext
enable
configure terminal
interface Loopback0
description Loopback-BR1-EDGE-R4
ip address 10.60.255.1 255.255.255.255
no shutdown
end
write
```
![](images/Pasted%20image%2020260111014957.png)

<br>

## **3.8 Interface Description Configuration - Headquarters**


Interface descriptions are applied on headquarters routing and switching devices to make link roles easy to recognize during configuration and troubleshooting. Descriptions follow a simple, consistent format that states the peer device and the purpose of the link.

Only internal headquarters devices are included in this step. ISP routers are treated as external provider equipment and are intentionally excluded from description configuration.



### **Port Description Mapping - Headquarters**

The table below defines the target descriptions for headquarters links. The same naming style is used across all headquarters devices.

| **Device**    | **Interface** | **Description**                  |
| ------------- | ------------- | -------------------------------- |
| HQ-EDGE-R3    | Gi0/1         | To HQ-CORE-R1 (P2P)              |
| HQ-EDGE-R3    | Gi0/2         | To HQ-CORE-R2 (P2P)              |
| HQ-EDGE-R3    | Gi0/5         | To HQ-ISP (WAN)                  |
| HQ-CORE-R1    | Gi0/1         | To HQ-EDGE-R3 (P2P)              |
| HQ-CORE-R1    | Gi0/5         | To HQ-CORE-R2 (Inter-core)       |
| HQ-CORE-R1    | Gi0/0         | To HQ-DIST-SW-01 (ROAS)          |
| HQ-CORE-R2    | Gi0/2         | To HQ-EDGE-R3 (P2P)              |
| HQ-CORE-R2    | Gi0/5         | To HQ-CORE-R1 (Inter-core)       |
| HQ-CORE-R2    | Gi0/0         | To HQ-DIST-SW-02 (ROAS)          |
| HQ-DIST-SW-01 | Gi0/0         | To HQ-CORE-R1 (trunk)            |
| HQ-DIST-SW-01 | Gi0/3         | To HQ-DIST-SW-02 (L2 Redundancy) |
| HQ-DIST-SW-01 | Gi0/2         | To HQ-ACC-SW-01 (trunk)          |
| HQ-DIST-SW-02 | Gi0/0         | To HQ-CORE-R2 (trunk)            |
| HQ-DIST-SW-02 | Gi0/1         | To HQ-DIST-SW-01 (L2 Redundancy) |
| HQ-DIST-SW-02 | Gi0/3         | To HQ-ACC-SW-02 (trunk)          |
| HQ-ACC-SW-01  | Gi0/1         | To HQ-DIST-SW-01 (trunk)         |
| HQ-ACC-SW-01  | Gi0/3         | To HQ-ACC-SW-02 (L2 Redundancy)  |
| HQ-ACC-SW-01  | Gi0/0         | To HQ-SRV-INFRA-01 (access port) |
| HQ-ACC-SW-01  | Gi0/2         | To HQ-SRV-MON-01 (access port)   |
| HQ-ACC-SW-02  | Gi0/2         | To HQ-DIST-SW-02 (trunk)         |
| HQ-ACC-SW-02  | Gi0/3         | To HQ-ACC-SW-01 (L2 Redundancy)  |
| HQ-ACC-SW-02  | Gi0/0         | To HQ-OFFICE-01 (access port)    |
| HQ-ACC-SW-02  | Gi0/1         | To HQ-FINANCE-01 (access port)   |


<br>

### **Configuration Example - HQ-CORE-R1**

Links:

- Gi0/1 -> HQ-EDGE-R3 Gi0/1 (P2P)
    
- Gi0/5 -> HQ-CORE-R2 Gi0/5 (Inter-core)
    
- Gi0/0 -> HQ-DIST-SW-01 Gi0/0 (ROAS)
    

Configuration:

```plaintext
enable
configure terminal
interface GigabitEthernet0/1
description To HQ-EDGE-R3 
exit
interface GigabitEthernet0/5
description To HQ-CORE-R2 
exit
interface GigabitEthernet0/0
description To HQ-DIST-SW-01 
exit
end
write memory
```
![](images/Pasted%20image%2020260110033335.png)

<br>

### **Configuration Example - HQ-DIST-SW-01**

Links:

- Gi0/0 -> HQ-CORE-R1 Gi0/0 (Trunk)
    
- Gi0/3 -> HQ-DIST-SW-02 Gi0/1 (L2 Redundancy)
    
- Gi0/2 -> HQ-ACC-SW-01 Gi0/1 (Trunk)
    

Configuration:

```plaintext
enable
configure terminal
interface GigabitEthernet0/0
description To HQ-CORE-R1 
exit
interface GigabitEthernet0/3
description To HQ-DIST-SW-02 
exit
interface GigabitEthernet0/2
description To HQ-ACC-SW-01 
exit
end
write memory
```
![](images/Pasted%20image%2020260110033503.png)

<br>

### **Verification**

Example verification on **HQ-CORE-R1**:

```plaintext
show interfaces description
```
![](images/Pasted%20image%2020260110033611.png)

<br>

### **Results**

Headquarters routing and switching interfaces include clear descriptions that identify peer devices and link purpose. Troubleshooting and verification steps can reference interface descriptions to confirm physical and logical connectivity more quickly.


<br>

## **3.9 Interface Description Configuration - Branch**


Interface descriptions are applied on branch routing and switching devices to clearly indicate link purpose and device connections. A consistent description format is used to simplify orientation in the topology and support troubleshooting.

Only internal branch devices are included in this section. The ISP router is treated as external provider equipment and is excluded from description configuration.



### **Port Description Mapping - Branch**

The table below defines interface descriptions for branch devices. Descriptions reflect the role of each link within the branch topology.

| **Device**     | **Interface** | **Description**                         |
| -------------- | ------------- | --------------------------------------- |
| BR1-EDGE-R4    | Gi0/1         | To BR1-DIST-SW-01 Gi0/1 (ROAS)          |
| BR1-EDGE-R4    | Gi0/2         | To BR1-DIST-SW-02 Gi0/2 (ROAS)          |
| BR1-EDGE-R4    | Gi0/5         | To BR1-ISP Gi0/5 (WAN)                  |
| BR1-DIST-SW-01 | Gi0/1         | To BR1-EDGE-R4 Gi0/1 (trunk)            |
| BR1-DIST-SW-01 | Gi0/0         | To BR1-DIST-SW-02 Gi0/0 (L2 Redundancy) |
| BR1-DIST-SW-01 | Gi0/2         | To BR1-OFFICE-01 e0 (access port)       |
| BR1-DIST-SW-01 | Gi0/3         | To BR1-SALES-01 e0 (access port)        |
| BR1-DIST-SW-02 | Gi0/2         | To BR1-EDGE-R4 Gi0/2 (trunk)            |
| BR1-DIST-SW-02 | Gi0/0         | To BR1-DIST-SW-01 Gi0/0 (L2 Redundancy) |
| BR1-DIST-SW-02 | Gi0/1         | To BR1-OFFICE-02 e0 (access port)       |
| BR1-DIST-SW-02 | Gi0/3         | To BR1-PRINTER-01 e0 (access port)      |

<br>

### **Configuration Example - BR1-EDGE-R4**

Links:

- Gi0/5 -> BR1-ISP Gi0/5 (WAN)
    
- Gi0/1 -> BR1-DIST-SW-01 Gi0/1 (ROAS)
    
- Gi0/2 -> BR1-DIST-SW-02 Gi0/2 (ROAS)
    

Configuration:

```plaintext
enable
configure terminal
interface GigabitEthernet0/5
description To BR1-ISP Gi0/5 
exit
interface GigabitEthernet0/1
description To BR1-DIST-SW-01 Gi0/1 
exit
interface GigabitEthernet0/2
description To BR1-DIST-SW-02 Gi0/2 
exit
end
write memory
```
![](images/Pasted%20image%2020260109001034.png)

<br>

### **Configuration Example - BR1-DIST-SW-01**

Links:

- Gi0/1 -> BR1-EDGE-R4 Gi0/1 (trunk)
    
- Gi0/0 -> BR1-DIST-SW-02 Gi0/0 (L2 Redundancy)
    
- Gi0/2 -> BR1-OFFICE-01 e0 (access port)
    
- Gi0/3 -> BR1-SALES-01 e0 (access port)
    

Configuration:

```plaintext
enable
configure terminal
interface GigabitEthernet0/1
description To BR1-EDGE-R4 Gi0/1 
exit
interface GigabitEthernet0/0
description To BR1-DIST-SW-02 Gi0/0 
exit
interface GigabitEthernet0/2
description To BR1-OFFICE-01 e0 
exit
interface GigabitEthernet0/3
description To BR1-SALES-01 e0 
exit
end
write memory
```
![](images/Pasted%20image%2020260109001238.png)

<br>

### **Verification**

Example verification on **BR1-EDGE-R4**:

```plaintext
show interfaces description
```
![](images/Pasted%20image%2020260109001116.png)

<br>

### **Results**

Branch interfaces include clear descriptions that identify peer devices and link roles (WAN, ROAS, trunk, redundancy, and access ports). This improves readability of the configuration and supports future Layer 2 and inter-VLAN routing tasks. Branch interface descriptions are applied using the same format as headquarters. The branch site is ready for Layer 2 configuration and inter-VLAN routing in the next chapters.



<br>

## **3.10 Connectivity Verification**


This last section verifies that basic Layer 3 connectivity between routing devices is working correctly. The checks confirm that WAN and point-to-point router links are up, IP addressing is applied as expected, and directly connected neighbors are reachable before continuing with Layer 2 and inter-VLAN routing configuration.



### **Router Interface Status**

Interface status is checked to confirm that configured router interfaces are operational and in the correct state.

Example: **BR1-EDGE-R4**

- Gi0/5 -> BR1-ISP Gi0/5 (89.203.0.2/30)

```plaintext
show ip interface brief
```
![](images/Pasted%20image%2020260109001615.png)

<br>

### **Ping Tests**

Ping tests verify basic reachability between directly connected devices.

#### **Headquarters – Core and Edge Connectivity**

![](images/Pasted%20image%2020260110032548.png)
    
- **HQ-CORE-R1 → HQ-CORE-R2** (inter-core link)
    

```plaintext
ping 10.50.200.10
```
![](images/Pasted%20image%2020260109001812.png)


- **HQ-CORE-R2 → HQ-CORE-R1** (inter-core link)
    
- **HQ-CORE-R2 → HQ-EDGE-R3** (core to edge)

```
ping 10.50.200.9
ping 10.50.200.5
```
![](images/Pasted%20image%2020260109001930.png)

<br>

#### **Branch-01 – WAN Connectivity**

    
- **BR1-EDGE-R4 → BR1-ISP**
    

```plaintext
ping 89.203.0.1
```
![](images/Pasted%20image%2020260109002005.png)

<br>

### **Neighbor Verification**

Neighbor visibility is checked to confirm correct physical cabling and Layer 2 adjacency between directly connected routing devices.

Example: **HQ-CORE-R1**

```plaintext
show cdp neighbors
```
![](images/Pasted%20image%2020260109002128.png)

<br>

### **Results**

Basic connectivity and neighbor visibility between **HQ-CORE-R1**, **HQ-CORE-R2**, **HQ-EDGE-R3**, and **BR1-EDGE-R4** toward their WAN peers are confirmed. All configured point-to-point links operate as expected, providing a stable baseline for Layer 2 configuration and inter-VLAN routing in the next chapters.

<br>

## **3.11 Conclusion**

The basic device configuration phase is complete. Infrastructure servers are assigned static IPv4 addresses, routing and switching devices use consistent hostnames, and WAN and inter-router point-to-point interfaces, including loopback interfaces, are configured. Interface descriptions are applied to clearly document link purpose and device connections, and basic interface status and direct connectivity are verified using ICMP and neighbor checks.

The topology is prepared for Layer 2 configuration, including VLAN creation, trunk links, and 802.1Q encapsulation, which are introduced in the next chapter.

<br>

---


<br>


**Next Chapter:** [Switching and VLAN Configuration](04-switching-and-vlan-configuration.md)
