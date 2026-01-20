
<br>

# **11 - Troubleshooting**


<br>

## **11.1 Introduction**

This final practical chapter documents troubleshooting scenarios that happened during the real development of the enterprise network project. Its purpose is to show a clear and structured way to identify, analyze, and fix configuration errors in a complex network environment.

The scenarios presented here are based on issues that appeared during real work, not on made-up examples. They reflect realistic problems that can affect routing, management access, or core network services.

A total of three independent problems are analyzed to confirm that the network returns to a stable and fully operational state after each fix.

<br>

## **11.2 Topology Diagram**

![](images/Pasted%20image%2020260120034004.png)


<br>

## **11.3 Objectives**

**This troubleshooting chapter focuses on three scenarios that appeared during the project:**

1. Verify and correct OSPF configuration on the GRE tunnel to restore inter-site connectivity.
    
2. Restore Internet access for headquarters VLANs by fixing missing default route propagation after NAT/PAT configuration.
    
3. Restore connectivity related to VLAN 99 by resolving routing issues that prevent communication between management devices and servers.

<br>

## **11.4 OSPF Area Mismatch on GRE Tunnel**

### Issue Overview

During testing, inter-site connectivity between Headquarters and Branch-01 is not fully operational. Branch clients are unable to obtain IP addresses via DHCP, and traffic from Headquarters servers does not reach Branch VLANs. At the same time, the GRE tunnel between edge routers remains up.



**The following observations are confirmed:**

- The GRE tunnel between HQ-EDGE-R3 and BR1-EDGE-R4 is UP/UP.
    
- IP connectivity between HQ-EDGE-R3 and BR1-EDGE-R4 over the tunnel works.
    
- DHCP operates correctly in Headquarters VLANs.
    
- DHCP server and helper-address configurations are correct.
    

Based on these observations, GRE configuration, IP addressing, and DHCP services are excluded as root causes. The issue is isolated to dynamic routing over the GRE tunnel.

<br>

### **1) Diagnostics**

#### OSPF Neighbor Status

```plaintext
show ip ospf neighbor
```
![](images/Pasted%20image%2020260119214729.png)

No OSPF neighbors are established on the GRE tunnel interface. The expected FULL adjacency state is missing, indicating a routing protocol issue.

#### OSPF Log Inspection

```plaintext
show logging | include OSPF
```
![](images/Pasted%20image%2020260119214750.png)

The router repeatedly reports OSPF errors indicating a mismatch between area IDs on the GRE tunnel.

<br>

### 2) Root Cause

**The GRE tunnel interface on BR1-EDGE-R4 is configured with an incorrect OSPF area ID. While the Headquarters edge router uses the backbone area (area 0), the Branch edge router is configured in a different area. Due to this mismatch, OSPF adjacency cannot be established and routing information is not exchanged between sites.**

<br>

### 3) Fix

#### Remove Incorrect OSPF Configuration

```plaintext
configure terminal
interface Tunnel0
no ip ospf 1 area 1
exit
```
![](images/Pasted%20image%2020260119214838.png)


#### Apply Correct OSPF Configuration

```plaintext
configure terminal
interface Tunnel0
ip ospf 1 area 0
exit
end
write memory
```
![](images/Pasted%20image%2020260119214930.png)


<br>

### 4) Verification

#### OSPF Neighbor Verification

```plaintext
show ip ospf neighbor
```
![](images/Pasted%20image%2020260119214946.png)

The OSPF neighbor on the GRE tunnel is now in the FULL state, confirming that routing adjacency is established.

<br>

#### DHCP Verification – Branch VLAN

* BR-SALES-01

```plaintext
ip dhcp
```
![](images/Pasted%20image%2020260119215036.png)

A Branch client successfully receives an IP address from the correct DHCP scope, confirming restored inter-site routing and relay functionality.

### Results

After correcting the OSPF area configuration on the GRE tunnel, dynamic routing between Headquarters and Branch-01 operates correctly. Inter-site connectivity is restored, DHCP functions as expected in Branch VLANs, and the network returns to a stable operational state.



<br>

## **11.5 HQ VLAN Traffic Cannot Reach Internet (NAT/PAT)**

### Issue Overview

After completing the NAT/PAT configuration on the Headquarters edge router, users in Headquarters VLANs are unable to reach external networks. ICMP traffic from internal VLANs fails when testing connectivity toward ISP-facing interfaces and Internet destinations.

**The following observations are confirmed:**

- NAT/PAT configuration on HQ-EDGE-R3 is present and unchanged.
    
- HQ-EDGE-R3 can successfully ping the ISP router.
    
- Inter-site connectivity between Headquarters and Branch remains functional.
    
- Internal routing between HQ VLANs operates correctly.
    

Based on these observations, NAT/PAT rules, ISP connectivity, and internal VLAN routing are excluded as root causes. The issue is isolated to routing information between core and edge routers.

<br>

### **1) Diagnostics**

#### Connectivity Tests from Headquarters VLAN 30 to ISP

* Ping from HQ-OFFICE-01 to HQ-ISP (GigabitEhernet0/5-WAN)

```plaintext
ping 37.48.0.2
```
![](images/Pasted%20image%2020260119222133.png)

Ping requests from Headquarters VLAN 30 fail, indicating that traffic does not reach the edge router where NAT/PAT is applied.

#### Routing Table Check on both HQ Core Routers 


* HQ-CORE-R1

```plaintext
show ip route
```
![](images/Pasted%20image%2020260119222343.png)

<br>

* HQ-CORE-R2

```plaintext
show ip route
```
![](images/Pasted%20image%2020260119222430.png)

The routing table on HQ core routers does not contain a default route (0.0.0.0/0). The output explicitly shows that the gateway of last resort is not set. As a result, traffic destined for external networks is dropped before reaching the edge router.
<br>

### **2) Root Cause**

Although a default route is configured on HQ-EDGE-R3 toward the ISP, it is not advertised into OSPF. As a result, HQ core routers have no knowledge of a path to external networks and cannot forward outbound traffic to the edge router for NAT/PAT processing.

<br>

### **3) Fix**

#### Advertise Default Route into OSPF on HQ Edge Router

```plaintext
enable
configure terminal
router ospf 1
default-information originate
end
write memory
```
![](images/Pasted%20image%2020260119222618.png)

This configuration injects the existing default route from HQ-EDGE-R3 into OSPF, allowing HQ core routers to dynamically learn the path to external networks.

### **4) Verification**

#### Routing Verification on HQ-CORE-R1

```plaintext
show ip route
```
![](images/Pasted%20image%2020260119222705.png)

The routing table now contains an OSPF-learned default route pointing toward HQ-EDGE-R3.

<br>

#### **Connectivity Verification from Headquarters VLAN 30**

* Ping from HQ-OFFICE-01 to HQ-ISP (GigabitEhernet0/5-WAN)

```plaintext
ping 37.48.0.2
```
![](images/Pasted%20image%2020260119225014.png)

Ping tests from Headquarters VLANs to external destinations succeed, confirming restored connectivity.

#### NAT/PAT Verification on HQ Edge Router

```
show ip nat translations
```
![](images/Pasted%20image%2020260119223027.png)

The output confirms that internal private IP addresses are being translated to the public address on HQ-EDGE-R3, verifying that NAT/PAT operates correctly after routing is restored.

### **Results**

After advertising the default route into OSPF, Headquarters VLAN traffic successfully reaches the edge router and is translated using NAT/PAT. Internet access from internal VLANs is restored, and the network operates as designed.


<br>

## **11.6 VLAN 99 Management Connectivity Failure (Return Path)**

### Issue Overview

During monitoring and management testing, servers are unable to reach network devices in the management VLAN (VLAN 99). ICMP traffic from servers to management IP addresses on switches fails, even though management VLAN connectivity within each site remains available.

**The following observations are confirmed:**

- Ping between management switches within VLAN 99 works.
    
- Core and edge routers can successfully ping management IP addresses on switches.
    
- Ping from servers to management switches in VLAN 99 fails consistently (Headquarters and Branch).
    

Based on these observations, VLAN configuration, IP addressing, and Layer 2 connectivity are excluded as root causes. The issue is isolated to Layer 3 return path routing on management switches.

<br>

### **1) Diagnostics**

#### Server-to-Switch Ping Failure (Example)

```plaintext
ping 10.60.99.50
```
![](images/Pasted%20image%2020260119225433.png)

Ping from the Infrastructure Server to the management IP address of BR1-DIST-SW-01 (10.60.99.50) fails, confirming that server-to-switch connectivity in VLAN 99 is broken!!!!!!

<br>

#### Branch Switch Routing Check

```plaintext
show ip route
```
![](images/Pasted%20image%2020260119225147.png)

The routing table does not contain a default route (0.0.0.0/0). Although Layer 3 routing is enabled on the switch, no route exists for destinations outside the management VLAN. As a result, the switch cannot return traffic to server subnets.

<br>

### **2) Root Cause**

**Layer 3 routing is enabled on the management switches, but a default route is missing. Without a default route pointing toward the site gateway in VLAN 99, the switches have no return path to server networks outside the management subnet. ICMP Echo Requests reach the switches, but Echo Replies cannot be routed back to the source servers.**

<br>

### **3) Fix**

#### Enable Layer 3 Routing and Configure Default Route (Branch Example)

```plaintext
configure terminal
ip routing
ip route 0.0.0.0 0.0.0.0 10.60.99.1
end
write memory
```
![](images/Pasted%20image%2020260119225531.png)

This configuration enables routing on the switch and installs a default route pointing to the site gateway in VLAN 99.


<br>

### **4) Verification**


**After applying the fix, the routing table is checked to confirm that a default route is present on the management switch.**

* BR1-DIST-SW-01

```
show ip route
```
![](images/Pasted%20image%2020260119225738.png)


**The routing table now shows a default route (0.0.0.0/0) pointing to 10.60.99.1. The gateway of last resort is set, confirming that the management switch has a valid return path to external networks.**



#### HQ-SRV-INFRA-01 Ping to BR1-DIST-SW-01 Verification (Example)

```plaintext
ping 10.60.99.50
```
![](images/Pasted%20image%2020260119225847.png)

Ping from the Infrastructure Server to BR1-DIST-SW-01 succeeds, confirming that return path routing is restored.

<br>

### Results

After configuring a default route on VLAN 99 management switches, return path routing is restored. Servers can successfully communicate with management devices across sites, enabling reliable monitoring and management access as designed.

<br>

## **11.7 Conclusion**

This final practical chapter of the enterprise project summarizes the resolution of key routing and connectivity issues identified during the network build. All problems were analyzed, fixed, and verified using a clear and structured troubleshooting process.

The next chapter provides a clear overview of all project chapters and explains their purpose and importance within the project design.

<br>

---


<br>

**Final Chapter:** [Conclusion and Summary](12-conclusion-and-summary.md)
