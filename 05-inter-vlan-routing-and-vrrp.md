
<br>

# **5 - Inter VLAN Routing and VRRP**

<br>


## **5.1 Introduction**

This chapter defines the Layer 3 design of the enterprise network used in Project 04. It builds on the existing VLAN and switching configuration and introduces inter-VLAN routing, gateway redundancy, and internal routing required for reliable network operation.

The headquarters site hosts critical infrastructure services and therefore implements gateway redundancy using VRRP to ensure continuous access during router or link failures. Inter-VLAN routing is provided using a router-on-a-stick (ROAS) design to enable controlled communication between VLANs.

Dynamic internal routing is implemented using OSPF. Headquarters core and edge routers operate in OSPF area 0 as the routing backbone, while the branch site is placed in a separate OSPF area to reflect a hierarchical enterprise design. Static routing is used on WAN links to provide basic connectivity and prepare the network for GRE tunnel deployment in the following chapter.

Management addressing for VLAN 99 is configured across all switches to support remote SSH access and Zabbix monitoring. The chapter concludes with verification and diagnostic checks that confirm correct Layer 3 operation and WAN connectivity.

<br>


## **5.2 Topology Diagram**

![](images/Pasted%20image%2020260110203821.png)


<br>

## **5.3 Objectives**

1. Enable inter-VLAN routing using a router-on-a-stick design to allow controlled Layer 3 communication between internal VLANs.
    
2. Configure OSPF for dynamic internal routing, using area 0 for the headquarters core and edge routers and a separate area for the branch site.
    
3. Implement VRRP on the headquarters core routers to provide a stable and redundant default gateway for critical VLANs.
    
4. Validate VRRP behavior by simulating a primary gateway failure and confirming automatic takeover by the backup router.
    
5. Configure static routing on WAN links between edge routers, ISP, and the Internet cloud as preparation for GRE tunnel deployment.
    
6. Configure VLAN 99 management IP addressing on all switches in the headquarters and branch sites.
    
7. Verify correct Layer 3 routing, OSPF neighbour connectivity


<br>


## **5.4 Inter-VLAN Routing - Router-on-a-Stick Configuration**


This first section configures inter-VLAN routing using a router-on-a-stick (ROAS) design at both the headquarters and branch sites. VLAN traffic is carried over 802.1Q trunks toward the routing devices, and each VLAN is terminated on a dedicated router subinterface.

Each subinterface uses 802.1Q encapsulation and an IP address within the VLAN subnet. The VRRP virtual IP address (VIP) is used as the default gateway for end devices and provides gateway redundancy at the headquarters site. VRRP configuration and failover testing are implemented in the next section.

<br>

### **Headquarters - ROAS Subinterface Overview**

| **VLAN ID** | **HQ-CORE-R1 Gi0/0 Subinterface** | **HQ-CORE-R2 Gi0/0 Subinterface** | **VRRP VIP (Gateway)** |
| ------: | ----------------------------- | ----------------------------- | ------------------ |
|      10 | Gi0/0.10                      | Gi0/0.10                      | 10.50.10.1         |
|      20 | Gi0/0.20                      | Gi0/0.20                      | 10.50.20.1         |
|      30 | Gi0/0.30                      | Gi0/0.30                      | 10.50.30.1         |
|      40 | Gi0/0.40                      | Gi0/0.40                      | 10.50.40.1         |
|      99 | Gi0/0.99                      | Gi0/0.99                      | 10.50.99.1         |


>**Notes.:** Each HQ Core Router uses a unique router subinterface IP address within each VLAN subnet (.2 and .3). The VRRP virtual IP (.1) is configured as the default gateway for all end devices.


<br>

### **Headquarters - Router Subinterface Configuration**

<br>

#### HQ-CORE-R1 (Trunk Interfaces Gi0/0 to HQ-DIST-SW-01)

```plaintext
enable
configure terminal
interface GigabitEthernet0/0
no shutdown
exit
interface GigabitEthernet0/0.10
encapsulation dot1Q 10
ip address 10.50.10.2 255.255.255.192
no shutdown
exit
interface GigabitEthernet0/0.20
encapsulation dot1Q 20
ip address 10.50.20.2 255.255.255.192
no shutdown
exit
interface GigabitEthernet0/0.30
encapsulation dot1Q 30
ip address 10.50.30.2 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/0.40
encapsulation dot1Q 40
ip address 10.50.40.2 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/0.99
encapsulation dot1Q 99
ip address 10.50.99.2 255.255.255.192
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020260110225407.png)

<br>

#### HQ-CORE-R2 (Trunk Interfaces Gi0/0 HQ-DIST-SW-02)

```plaintext
enable
configure terminal
interface GigabitEthernet0/0
no shutdown
exit
interface GigabitEthernet0/0.10
encapsulation dot1Q 10
ip address 10.50.10.3 255.255.255.192
no shutdown
exit
interface GigabitEthernet0/0.20
encapsulation dot1Q 20
ip address 10.50.20.3 255.255.255.192
no shutdown
exit
interface GigabitEthernet0/0.30
encapsulation dot1Q 30
ip address 10.50.30.3 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/0.40
encapsulation dot1Q 40
ip address 10.50.40.3 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/0.99
encapsulation dot1Q 99
ip address 10.50.99.3 255.255.255.192
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020260110225527.png)

<br>

### **Verification**

Verification is performed on both core routers. The example below shows the verification workflow on HQ-CORE-R1 only.

##### HQ-CORE-R1 (Example)

```plaintext
show ip interface brief
```
![](images/Pasted%20image%2020260110225703.png)

<br>


### **Branch-01 - ROAS Port and Gateway Overview**

<br>

#### **Branch-01 VLAN Gateway Mapping with Subinterface**

| **VLAN ID** | **VLAN Name** | **Routed on BR1-EDGE-R4** | **Subinterface** | **Default Gateway** |
| ----------: | ------------- | ------------------------- | ---------------- | ------------------- |
|          50 | Office-01     | Gi0/1                     | Gi0/1.50         | 10.60.50.1          |
|          60 | Office-02     | Gi0/2                     | Gi0/2.60         | 10.60.60.1          |
|          70 | Sales         | Gi0/1                     | Gi0/1.70         | 10.60.70.1          |
|          80 | Printer       | Gi0/2                     | Gi0/2.80         | 10.60.80.1          |
|          99 | Management    | Gi0/1                     | Gi0/1.99         | 10.60.99.1          |

#### **Trunk Use Summary**

|Trunk on BR1-EDGE-R4|VLANs carried (routed on this trunk)|
|---|---|
|Gi0/1|50, 70, 99|
|Gi0/2|60, 80|

>**Notes.:** Branch-01 uses a single edge router with ROAS split across two trunk links. Each VLAN is routed on exactly one trunk subinterface. Gateway redundancy is implemented only at the headquarters site.


<br>

### **Branch-01 - Router Subinterface Configuration**

<br>

#### BR1-EDGE-R4 (Trunk Interface Gi0/1 to BR1-DIST-SW-01)

```
enable
configure terminal
interface GigabitEthernet0/1
no shutdown
exit
interface GigabitEthernet0/1.50
encapsulation dot1Q 50
ip address 10.60.50.1 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/1.70
encapsulation dot1Q 70
ip address 10.60.70.1 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/1.99
encapsulation dot1Q 99
ip address 10.60.99.1 255.255.255.192
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020260110225734.png)

<br>

#### BR1-EDGE-R4 (Trunk Interface Gi0/2 to BR1-DIST-SW-02)

```
enable
configure terminal
interface GigabitEthernet0/2
no shutdown
exit
interface GigabitEthernet0/2.60
encapsulation dot1Q 60
ip address 10.60.60.1 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/2.80
encapsulation dot1Q 80
ip address 10.60.80.1 255.255.255.224
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020260110225751.png)

<br>

### **Verification**

##### **BR1-EDGE-R4 (Example)**

```
show ip interface brief
```
![](images/Pasted%20image%2020260110225902.png)

<br>

### **Results (HQ and Branch)**

Inter-VLAN routing runs at both sites using a router-on-a-stick design. VLAN traffic uses 802.1Q trunks, and each VLAN uses a router subinterface as its default gateway.

At headquarters, VLAN subinterfaces are up and ready for the next section (OSPF). At the branch site, BR1-EDGE-R4 provides working gateways for the branch VLANs, including user VLANs, the printer VLAN, and the management VLAN.

All required subinterfaces exist and operate in an up/up state, so the network is ready for OSPF configuration.


<br>

## **5.5 OSPF - Internal Routing Configuration**


## OSPF - Internal Routing Design Overview

### Introduction

This configuration part enables dynamic internal routing using OSPF across the enterprise network.

Headquarters uses OSPF area 0 as the backbone. This keeps routing centralized and stable at the main site and follows common enterprise design practices. Loopback interfaces on the headquarters core routers are advertised into OSPF area 0 to provide stable router identification and reliable management reachability.

The branch site uses a separate OSPF area (area 1). This reduces routing complexity at the branch and keeps a clear hierarchical structure between the core and the branch. The branch edge router also advertises its loopback interface into OSPF to ensure consistent identification and reachability independent of physical links.

OSPF is an advanced function for this design because it supports:

- automatic route learning inside the network
    
- fast convergence when a link or router fails
    
- scalable routing between multiple routers and sites
    
- dynamic route exchange over the GRE tunnel in the next chapter

<br>



### **Headquarters - OSPF Area 0 Configuration**

<br>

#### HQ-CORE-R1 (OSPF area 0)

```plaintext
enable
configure terminal
router ospf 1
router-id 1.1.1.1
network 10.50.200.0 0.0.0.3 area 0
network 10.50.200.8 0.0.0.3 area 0
network 10.50.10.0 0.0.0.63 area 0
network 10.50.20.0 0.0.0.63 area 0
network 10.50.30.0 0.0.0.255 area 0
network 10.50.40.0 0.0.0.255 area 0
network 10.50.99.0 0.0.0.63 area 0
network 10.50.255.1 0.0.0.0 area 0
exit
end
write
```
![](images/Pasted%20image%2020260110225931.png)

<br>

#### HQ-CORE-R2 (OSPF area 0)

```plaintext
enable
configure terminal
router ospf 1
router-id 2.2.2.2
network 10.50.200.4 0.0.0.3 area 0
network 10.50.200.8 0.0.0.3 area 0
network 10.50.10.0 0.0.0.63 area 0
network 10.50.20.0 0.0.0.63 area 0
network 10.50.30.0 0.0.0.255 area 0
network 10.50.40.0 0.0.0.255 area 0
network 10.50.99.0 0.0.0.63 area 0
network 10.50.255.2 0.0.0.0 area 0
exit
end
write
```
![](images/Pasted%20image%2020260110230001.png)

<br>

#### HQ-EDGE-R3 (OSPF area 0)

```plaintext
enable
configure terminal
router ospf 1
router-id 3.3.3.3
network 10.50.200.0 0.0.0.3 area 0
network 10.50.200.4 0.0.0.3 area 0
exit
end
write
```
![](images/Pasted%20image%2020260110230023.png)

<br>

### **Headquarters - Core Router Loopbacks in OSPF Area 0**


Loopback interfaces on the headquarters core routers and edge are advertised into OSPF area 0 to provide a stable router identity and reliable management access or for Zabbix monitoring. Because loopbacks are independent of physical interfaces, they remain reachable as long as the router is running.

### **Configuration**

#### HQ-CORE-R1

```plaintext
enable
configure terminal
router ospf 1
router-id 10.50.255.1
network 10.50.255.1 0.0.0.0 area 0
exit
end
write
```
![](images/Pasted%20image%2020260111015638.png)

<br>

#### HQ-CORE-R2

```plaintext
enable
configure terminal
router ospf 1
router-id 10.50.255.2
network 10.50.255.2 0.0.0.0 area 0
exit
end
write
```
![](images/Pasted%20image%2020260111015656.png)



#### HQ-EDGE-R3

```plaintext
enable
configure terminal
router ospf 1
router-id 10.50.255.3
network 10.50.255.3 0.0.0.0 area 0
end
write
```
![](images/Pasted%20image%2020260111033819.png)
<br>

### **Verification and Diagnostics**

**HQ-CORE-R1 (Example)**

```plaintext
show ip ospf neighbor
show ip route ospf
show ip ospf interface brief
```
![](images/Pasted%20image%2020260111034025.png)

<br>

### **Branch Configuration (Area 1)**


The branch edge router advertises all branch internal networks into OSPF and connects to headquarters through a hierarchical OSPF design.

The branch operates as a separate OSPF area to keep routing simple and structured, while the headquarters backbone remains centralized and stable.


#### BR1-EDGE-R4 (OSPF area 1)

```plaintext
enable
configure terminal
router ospf 1
router-id 4.4.4.4
network 10.60.50.0 0.0.0.255 area 1
network 10.60.60.0 0.0.0.255 area 1
network 10.60.70.0 0.0.0.255 area 1
network 10.60.80.0 0.0.0.31 area 1
network 10.60.99.0 0.0.0.63 area 1
exit
end
write
```
![](images/Pasted%20image%2020260110230630.png)

<br>
## Branch-01 - Loopback Advertisement into OSPF Area 1


The loopback interface on the Branch-01 edge router is explicitly advertised into OSPF area 1. This ensures stable router identification and consistent reachability for management and routing purposes, independent of physical interface state.

### Configuration

#### BR1-EDGE-R4

```plaintext
enable
configure terminal
router ospf 1
network 10.60.255.1 0.0.0.0 area 1
exit
end
write
```
![](images/Pasted%20image%2020260111020416.png)


<br>

### **OSPF Advanced verification - Wireshark Monitoring**

##### **Capture Location:**

- Routed point-to-point link between **HQ-EDGE-R3 Gi0/1** and **HQ-CORE-R1 Gi0/1**
    
![](images/Pasted%20image%2020260110181205.png)

### Wireshark Filter

```plaintext
ospf
```
![](images/Pasted%20image%2020260110230808.png)


Wireshark monitoring confirms a valid OSPF neighbor relationship between HQ-EDGE-R3 and HQ-CORE-R1 on the routed link. OSPF Hello packets are exchanged regularly using the multicast address 224.0.0.5, indicating stable control-plane operation and correct internal routing adjacency.

<br>

### **Results (HQ and Branch)**

OSPF provides dynamic routing between headquarters and the branch site. Headquarters operates as the OSPF backbone in area 0. The branch site uses area 1 to advertise its internal VLAN networks. The branch edge router exchanges routing information with the headquarters edge router over the GRE tunnel. Core routers at headquarters learn branch routes through the edge router without direct OSPF neighbor relationships.



<br>

## **5.6 VRRP - Gateway Redundancy Configuration**



VRRP provides a stable and redundant default gateway for internal VLANs at the headquarters site. End devices use a virtual IP address as their default gateway, while two core routers share the gateway role. If the active router becomes unavailable, the standby router takes over automatically.

<br>

### VRRP Design Overview

- VRRP is configured on both headquarters core routers.
    
- VRRP runs on all VLAN subinterfaces used for internal routing.
    
- A single virtual IP address (VIP) is used as the default gateway per VLAN.
    
- HQ-CORE-R1 operates as the primary gateway (higher priority).
    
- HQ-CORE-R2 operates as the standby gateway (lower priority).
    
- Preemption is enabled to restore the primary gateway role after recovery.
    

<br>

### **VRRP VIP (Default Gateway) Overview**

|VLAN ID|VRRP Group|VRRP VIP (Gateway)|R1 IP Address|R2 IP Address|
|--:|--:|--:|--:|--:|
|10|10|10.50.10.1|10.50.10.2|10.50.10.3|
|20|20|10.50.20.1|10.50.20.2|10.50.20.3|
|30|30|10.50.30.1|10.50.30.2|10.50.30.3|
|40|40|10.50.40.1|10.50.40.2|10.50.40.3|
|99|99|10.50.99.1|10.50.99.2|10.50.99.3|

<br>

### VRRP Configuration

#### HQ-CORE-R1 (Primary)

```plaintext
enable
configure terminal
interface GigabitEthernet0/0.10
vrrp 10 ip 10.50.10.1
vrrp 10 priority 110
vrrp 10 preempt
exit
interface GigabitEthernet0/0.20
vrrp 20 ip 10.50.20.1
vrrp 20 priority 110
vrrp 20 preempt
exit
interface GigabitEthernet0/0.30
vrrp 30 ip 10.50.30.1
vrrp 30 priority 110
vrrp 30 preempt
exit
interface GigabitEthernet0/0.40
vrrp 40 ip 10.50.40.1
vrrp 40 priority 110
vrrp 40 preempt
exit
interface GigabitEthernet0/0.99
vrrp 99 ip 10.50.99.1
vrrp 99 priority 110
vrrp 99 preempt
exit
end
write
```
![](images/Pasted%20image%2020260110230855.png)


<br>

#### HQ-CORE-R2 (Standby)

```plaintext
enable
configure terminal
interface GigabitEthernet0/0.10
vrrp 10 ip 10.50.10.1
vrrp 10 priority 100
exit
interface GigabitEthernet0/0.20
vrrp 20 ip 10.50.20.1
vrrp 20 priority 100
exit
interface GigabitEthernet0/0.30
vrrp 30 ip 10.50.30.1
vrrp 30 priority 100
exit
interface GigabitEthernet0/0.40
vrrp 40 ip 10.50.40.1
vrrp 40 priority 100
exit
interface GigabitEthernet0/0.99
vrrp 99 ip 10.50.99.1
vrrp 99 priority 100
exit
end
write
```
![](images/Pasted%20image%2020260110230948.png)

<br>

### **Verification and Diagnostics**

HQ-CORE-R1 

```plaintext
show vrrp brief
```
![](images/Pasted%20image%2020260110231103.png)


HQ-CORE-R2 

```plaintext
show vrrp brief
```
![](images/Pasted%20image%2020260110231244.png)

>**Notes.:** For detail show diagnostic is also command `show vrrp`

### Results

VRRP provides a consistent default gateway for all internal VLANs at the headquarters site. HQ-CORE-R1 operates as the active gateway, while HQ-CORE-R2 remains in standby state. If the primary router becomes unavailable, the standby router takes over the gateway role automatically, keeping network access available for end devices.


<br>

## **5.7 VRRP Failover Test - Primary Router Failure**


This section verifies VRRP gateway redundancy during a simulated failure. The primary core router (HQ-CORE-R1) is intentionally shut down to confirm that the standby router can take over the default gateway role without manual intervention.

<br>

### **Purpose**

The goal of this test is to confirm that VRRP provides high availability for default gateway services. When the primary router becomes unavailable, the backup router must immediately assume the gateway role so that client connectivity remains available.


<br>

### **Failure Simulation**

The VRRP failover is triggered by powering off the primary core router.

Action:

- HQ-CORE-R1 is shut down (powered off) in the topology.
    

This simulates an unexpected core router outage.


![](images/Pasted%20image%2020260110231404.png)


<br>

### **Verification on Backup Router (HQ-CORE-R2)**

After HQ-CORE-R1 is offline, HQ-CORE-R2 transitions from standby to active VRRP state as Master.

```plaintext
show vrrp brief
```
![](images/Pasted%20image%2020260110231434.png)

The output confirms that HQ-CORE-R2 now owns the VRRP virtual IP addresses and operates as the active default gateway for all VLANs.

<br>

### **Result**

VRRP failover operates as expected. When the primary core router is shut down, the standby core router automatically becomes the active VRRP gateway. Default gateway availability is preserved, and no manual configuration changes are required.

<br>

## **5.8 Static Routing - WAN and Internet-Cloud (GRE Preparation)**


**This routing part prepares the WAN and Internet connectivity required for reliable site-to-site communication.**

Static routes are used to clearly define how traffic flows between enterprise edge routers, ISP routers, and the Internet-Cloud. This ensures predictable reachability across WAN links and creates a stable foundation for the upcoming GRE tunnel configuration. Static routes are configured to ensure reachability between headquarters, branch, ISP routers, and the Internet-Cloud. 

This routing layer provides a simple and predictable path for GRE tunnel endpoints and enables external access from the Remote-Admin site.

<br>

### **Design Purpose**

- Ensure WAN reachability between edge routers across the simulated Internet.
    
- Allow ISP routers to forward traffic toward the Internet-Cloud.
    
- Configure the Internet-Cloud as a transit device with explicit routes to ISP WAN blocks.
    
- Provide a stable foundation for GRE tunnel operation and later remote administration access.
    

<br>

### HQ-EDGE-R3 (Headquarters Edge Router)

* Traffic from the headquarters edge router is forwarded toward the HQ ISP router for all external destinations.

```plaintext
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 37.48.0.1
end
write
```
![](images/Pasted%20image%2020260110231540.png)

<br>

### HQ-ISP (Headquarters ISP Router)

* Traffic from the HQ ISP router is forwarded toward the Internet-Cloud upstream.

```plaintext
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 46.23.10.2
end
write
```
![](images/Pasted%20image%2020260110231603.png)

<br>

### **Internet-Cloud**

The Internet-Cloud operates as a transit device. It uses explicit static routes for WAN address blocks behind each ISP router.

* Traffic toward headquarters WAN addresses is forwarded to the HQ ISP router.

* Traffic toward branch WAN addresses is forwarded to the Branch ISP router.

```plaintext
enable
configure terminal
ip route 37.48.0.0 255.255.255.252 46.23.10.1
ip route 89.203.0.0 255.255.255.252 93.184.216.1
end
write
```
![](images/Pasted%20image%2020260110231625.png)

Traffic toward branch WAN addresses is forwarded to the Branch ISP router.

<br>

### **BR1-EDGE-R4 (Branch Edge Router)**

* Traffic from the branch edge router is forwarded toward the Branch ISP router for all external destinations.

```plaintext
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 89.203.0.1
end
write
```
![](images/Pasted%20image%2020260110231650.png)

<br>

### **BR1-ISP (Branch ISP Router)**

* Traffic from the Branch ISP router is forwarded toward the Internet-Cloud upstream.

```plaintext
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 93.184.216.2
end
write
```
![](images/Pasted%20image%2020260110231724.png)


<br>

### **Verification**

The following tests confirm correct WAN reachability required for GRE tunnel operation.


<br>

##### **HQ-EDGE-R3**

Traffic direction:

    
- HQ-EDGE-R3 to Internet-Cloud (over ISP)
    

```plaintext
ping 46.23.10.2
```
![](images/Pasted%20image%2020260110231747.png)

<br>

##### **HQ-ISP**

Traffic direction:

* HQ-ISP to Internet-Cloud (WAN-uplink)
    

```plaintext
ping 46.23.10.2
```
![](images/Pasted%20image%2020260110231815.png)

<br>

##### **BR1-EDGE-R4**

Traffic direction:

    
- BR1-EDGE-R4 to Internet-Cloud (over ISP)
    

```plaintext
ping 93.184.216.2
```
![](images/Pasted%20image%2020260110231833.png)

<br>

##### **BR1-ISP**

Traffic direction:

- BR1-ISP to Internet-Cloud (WAN-uplink)
    
    

```plaintext
ping 93.184.216.2
```
![](images/Pasted%20image%2020260110231844.png)

<br>

### Result

WAN and Internet static routing is fully operational at both sites.
Connectivity across ISP routers and the Internet-Cloud is established, providing predictable end-to-end reachability. This routing layer fully prepares inter-site connectivity and creates a stable foundation for the upcoming GRE tunnel.




<br>

## **5.9 VLAN 99 - Switch Management IP Addressing**


VLAN 99 provides a dedicated management network for switches at both sites. Each switch uses an SVI on VLAN 99 for management access, centralized monitoring and Zabbix connectivity. End devices do not use VLAN 99, which keeps management and monitoring traffic separated from user data.


### **Objectives**

- Configure VLAN 99 SVI management IP addressing on all switches.
    
- Use the correct default gateway for management traffic at each site.
    
- Verify basic management reachability inside each site.
    

<br>

### **Addressing Reference**

Headquarters (HQ) VLAN 99

- Network: 10.50.99.0/26
    
- Management default gateway (VRRP VIP): 10.50.99.1
    
- Example switch IP: HQ-ACC-SW-01 = 10.50.99.30/26
    

Branch-01 (BR1) VLAN 99

- Network: 10.60.99.0/26
    
- Management default gateway (BR1-EDGE-R4): 10.60.99.1
    
- Example switch IP: BR1-DIST-SW-02 = 10.60.99.60/26
    

<br>

### **HQ Example - HQ-ACC-SW-01 (VLAN 99 SVI)**

Use VLAN 99 for management SVI and set the VRRP VIP as the default gateway.

```plaintext
enable
configure terminal
vlan 99
name Management
exit
interface vlan 99
description VLAN-99-HQ-ACC-SW-01 
ip address 10.50.99.30 255.255.255.192
no shutdown
exit
ip default-gateway 10.50.99.1
exit
write
```
![](images/Pasted%20image%2020260110231940.png)


<br>

### **Branch Example - BR1-DIST-SW-02 (VLAN 99 SVI)**

Use VLAN 99 for management SVI and set BR1-EDGE-R4 as the default gateway.

```plaintext
enable
configure terminal
vlan 99
name Management
exit
interface vlan 99
description VLAN-99-BR-1-DIST-SW-02
ip address 10.60.99.60 255.255.255.192
no shutdown
exit
ip default-gateway 10.60.99.1
exit
write
```
![](images/Pasted%20image%2020260110232017.png)

<br>

### **Verification**

HQ-ACC-SW-01

Traffic direction:

- HQ access switch to HQ management gateway (VRRP VIP)
    
* HQ acess switch ping to HQ-DIST-SW-01

```plaintext
ping 10.50.99.1
ping 10.50.99.10
```
![](images/Pasted%20image%2020260110234010.png)

<br>

BR1-DIST-SW-02

Traffic direction:

- Branch distribution switch to branch management gateway
    
- Branch distribution switch to peer distribution switch (VLAN 99)
    

```plaintext
ping 10.60.99.1
ping 10.60.99.50
```
![](images/Pasted%20image%2020260110233817.png)

<br>

### **Result**

VLAN 99 management interfaces are reachable on all switches. Management traffic is correctly routed at both sites, and switches can communicate with their local gateways and peer switches within the management VLAN. This provides a stable foundation for centralized and remote network administration.

<br>

## **VLAN 99 - Switch Routing and Default Route (HQ / Branch)**

This next section explains a routing issue found during Zabbix monitoring setup. Management switches had VLAN 99 SVI configured but no Layer 3 routing, so traffic from switches could not return to servers. Enabling routing and adding a default route fixes the communication.


### Configuration - Headquarters Switches (VLAN 99 Gateway: 10.50.99.1)

The following configuration is applied to all HQ distribution and access switches that use VLAN 99 for management.

```
configure terminal
ip routing
ip route 0.0.0.0 0.0.0.0 10.50.99.1
end
write memory
```



### Configuration - Branch Switches (VLAN 99 Gateway: 10.60.99.1)

Branch switches use a different management subnet and gateway. The same logic applies, with a different next-hop address.

```
configure terminal
ip routing
ip route 0.0.0.0 0.0.0.0 10.60.99.1
end
write memory
```



### Results

- Management switches can communicate with servers outside VLAN 99
    
- Zabbix can successfully monitor switches via SNMP
    
- Original HQ and Branch ACL policies can be safely re-applied
    
- No changes are required on core routers

>Note: With `ip routing` enabled, the `ip default-gateway` command is ignored. The switch now uses `ip route 0.0.0.0 0.0.0.0` for all external communication. Original gateway settings remain as a backup only.

<br>

## **5.10 Verification - OSPF Neighbors and Internal Reachability (HQ)**


**This verification covers headquarters only.**

- OSPF neighbor adjacency between HQ-CORE-R1, HQ-CORE-R2, and HQ-EDGE-R3
    
- Basic reachability tests from HQ servers to key routed links (core-to-core and edge-to-core)
    


<br>

### **OSPF Neighbor Verification**

#### **HQ-CORE-R1**

```plaintext
enable
show ip ospf neighbor
```
![](images/Pasted%20image%2020260111021539.png)

<br>

#### **HQ-CORE-R2**

```plaintext
enable
show ip ospf neighbor
```
![](images/Pasted%20image%2020260111021612.png)


*The OSPF neighbor tables show that both headquarters core routers use loopback-based router IDs (10.50.255.1 and 10.50.255.2) for their neighbor relationships. The original router IDs (1.1.1.1 and 2.2.2.2) are no longer visible because OSPF was restarted and now correctly uses the loopback addresses as the active router IDs.*

<br>

#### **HQ-EDGE-R3**

```plaintext
enable
show ip ospf neighbor
```
![](images/Pasted%20image%2020260110234138.png)



---


<br>

### **Ping Verification (HQ Servers)**

Targets are limited to routed point-to-point links:

- **Inter-core routed link (HQ-CORE-R1 <-> HQ-CORE-R2)**
    
    - HQ-CORE-R1 Gi0/5: 10.50.200.9
        
    - HQ-CORE-R2 Gi0/5: 10.50.200.10
        
- **Edge-to-core routed link (HQ-EDGE-R3 <-> HQ-CORE-R1)**
    
    - HQ-EDGE-R3 Gi0/1: 10.50.200.1
        
    - HQ-CORE-R1 Gi0/1: 10.50.200.2
        


<br>

#### **HQ-SRV-INFRA-01 (10.50.10.10)**

```plaintext
ping 10.50.200.9
ping 10.50.200.10
ping 10.50.200.1
ping 10.50.200.2
```
![](images/Pasted%20image%2020260110234323.png)

<br>

#### **HQ-SRV-MON-01 (10.50.20.10)**

```plaintext
ping 10.50.200.9
ping 10.50.200.10
ping 10.50.200.1
ping 10.50.200.2
```
![](images/Pasted%20image%2020260110234446.png)

<br>

### **Result**

OSPF neighbors form correctly between the HQ routers, and HQ servers reach the main routed links. This confirms that internal routing at headquarters is stable and ready for the next GRE tunnel chapter.


<br>

### **5.11 Conclusion**

This chapter finalized the internal Layer 3 network design. Inter-VLAN routing was configured using router-on-a-stick, OSPF provides dynamic internal routing, and VRRP ensures a stable default gateway at the headquarters. Static WAN routing and Internet-Cloud transit routing established reliable connectivity between edge routers, ISP routers, and the simulated internet. VLAN 99 enables centralized management access across the network.

All routing components are now correctly prepared for GRE tunnel deployment. Internal routing is stable, WAN connectivity is verified, and the network is ready for site-to-site connectivity between the headquarters and branch in the next chapter.

<br>

----

<br>

**Next Chapter:** [GRE Tunnels and Site Connectivity](06-gre-tunnels-and-site-connectivity.md)
