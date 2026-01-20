
<br>

# **6 - Gre Tunnels and Site Connectivity**

<br>

## **6.1 Introduction**


This chapter introduces the GRE (*Generic Routing Encapsulation*) tunnel used to connect the Headquarters and Branch-01 sites over the simulated Internet. The GRE tunnel creates a logical point-to-point link between edge routers, allowing internal networks at both sites to communicate as if they were directly connected.

In this project, the GRE tunnel is used to extend internal routing across WAN limits without opening internal VLAN addressing to the ISP or Internet-Cloud. All inter-site traffic is encapsulated inside GRE, while the underlying WAN infrastructure remains simple and based on static routing.

The GRE tunnel also prepares the network for advanced features used in later chapters. By running dynamic routing over the tunnel, centralized services such as DHCP, monitoring, and management can be shared between sites, enabling a scalable and enterprise-style site-to-site connectivity design.

<br>

## **6.2 Topology Diagram**

![](images/Pasted%20image%2020260111011944.png)


<br>


## **6.3 Objectives**


1. Configure a GRE (*Generic Routing Encapsulation*) tunnel between the headquarters and branch edge routers.
    
2. Monitoring GRE tunnel operation and verify encapsulation using basic diagnostic tools.
    
3. Verify end-to-end connectivity between headquarters and branch sites through the GRE tunnel.

<br>

## **6.4 Gre tunnels - Tunnel and OSPF setup**


Gre Tunnels function creates a virtual point-to-point link between Headquarters and Branch over the WAN.

In this project, the Gre tunnel acts like a private “path” between the two edge routers. It lets internal networks from both sites reach each other, even though the routers are separated by ISP and Internet-Cloud devices.

This setup prepares stable site-to-site connectivity for the next chapters (routing exchange, services, and later DHCP reachability across sites).

<br>

### **Tunnel overview**

| **Device**  | **Tunnel interface** | **Tunnel IP** | **Tunnel source (WAN IP)** | **Tunnel destination (WAN IP)** | **Assigned network** |
| ----------- | -------------------- | ------------- | -------------------------- | ------------------------------- | -------------------- |
| HQ-EDGE-R3  | Tunnel0              | 10.254.0.1/30 | 37.48.0.2                  | 89.203.0.2                      | 10.254.0.0/30        |
| BR1-EDGE-R4 | Tunnel0              | 10.254.0.2/30 | 89.203.0.2                 | 37.48.0.2                       | 10.254.0.0/30        |

<br>

### HQ-EDGE-R3 - Gre tunnel configuration

```plaintext
enable
configure terminal
interface Tunnel0
description GRE to BR1-EDGE-R4
ip address 10.254.0.1 255.255.255.252
tunnel source 37.48.0.2
tunnel destination 89.203.0.2
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020260110234602.png)

<br>

### BR1-EDGE-R4 - Gre tunnel configuration

```plaintext
enable
configure terminal
interface Tunnel0
description GRE to HQ-EDGE-R3
ip address 10.254.0.2 255.255.255.252
tunnel source 89.203.0.2
tunnel destination 37.48.0.2
no shutdown
exit
end
write
```
![](images/Pasted%20image%2020260110234636.png)


<br>

### **OSPF - Enable routing exchange over the tunnel**

OSPF is enabled on the tunnel interface so routes can be shared across the Gre link. This allows both sites to learn each other’s internal networks without adding many manual routes.

<br>

#### HQ-EDGE-R3 - OSPF on Tunnel0 (area 0)

```plaintext
enable
configure terminal
interface Tunnel0
ip ospf 1 area 0
end
write
```
![](images/Pasted%20image%2020260110234658.png)

<br>

#### BR1-EDGE-R4 - OSPF on Tunnel0 (area 0)

```plaintext
enable
configure terminal
interface Tunnel0
ip ospf 1 area 0
end
write
```
![](images/Pasted%20image%2020260111005501.png)

>**Notes.:** In OSPF routing, Area 0 is the backbone used for the Headquarters (HQ). Our Edge-R4 router is configured in Area 1 for local branch traffic, but its GRE Tunnel must stay in Area 0 to match the Headquarters. This setup makes Edge-R4 an Area Border Router (ABR) and creates a logical bridge between both sites. The meaning of this design is to allow the branch to communicate with the backbone while keeping local traffic isolated in Area 1. Without Area 0 on the tunnel interface, the branch cannot see the HQ networks and routing will fail.


<br>


### **Result**

The Gre tunnel is successfully established between both edge routers using the WAN infrastructure.

Tunnel interfaces are operational and provide a dedicated overlay network for inter-site traffic.

<br>

## **6.5 Gre Tunnel Monitoring with Wireshark**


This next section verifies GRE tunnel operation by monitoring encapsulated traffic on the WAN link. Wireshark is used to confirm that inter-site packets are correctly encapsulated inside GRE and transported across the Internet-facing infrastructure.

Monitoring GRE traffic provides a clear confirmation that the tunnel is active at Layer 3 and that routing traffic between Headquarters and Branch is passing through the overlay tunnel rather than the physical WAN directly.

<br>

### **Capture Location**

Traffic capture is performed on the WAN link between the edge router and the ISP router.

- **HQ site: HQ-EDGE-R3 ↔ HQ-ISP**
    
- **Branch site: BR1-EDGE-R4 ↔ BR1-ISP**
    

Capturing on the WAN interface allows visibility into GRE encapsulation before traffic enters the ISP and Internet-Cloud path.

<br>

### **Wireshark Filter**

The following display filter is used to identify GRE packets:

![](images/Pasted%20image%2020260110234941.png)
```plaintext
ospf
```
![](images/Pasted%20image%2020260110234846.png)

<br>


![](images/Pasted%20image%2020260110235040.png)
```plaintext
ospf
```
![](images/Pasted%20image%2020260110235029.png)

This filter shows GRE-encapsulated packets exchanged between the WAN IP addresses of both edge routers.

<br>

### **Result**

GRE encapsulated traffic is visible on the WAN link even without manual testing. OSPF Hello packets are sent regularly across the GRE tunnel and appear as GRE (protocol 47) traffic in Wireshark. This confirms that the tunnel is active at Layer 3 and that dynamic routing control traffic is successfully transported through the GRE overlay.


<br>

## **6.6 GRE Tunnel Connectivity Verification**

This section verifies end-to-end connectivity over the GRE tunnel after tunnel and OSPF configuration. The goal is to confirm that both sites can reach each other through the GRE overlay and that internal hosts can communicate across sites using routed paths.

<br>

### **1 - Edge Router to Edge Router (GRE Tunnel)**

These tests confirm that the GRE tunnel endpoints are reachable and that OSPF routing over the tunnel is operational.

<br>

#### HQ-EDGE-R3 → BR1-EDGE-R4 (GRE tunnel destination)

```plaintext
ping 89.203.0.2
```
![](images/Pasted%20image%2020260110235130.png)

<br>

#### BR1-EDGE-R4 → HQ-EDGE-R3 (GRE tunnel destination)

```plaintext
ping 37.48.0.2
```
![](images/Pasted%20image%2020260110235227.png)

Successful replies confirm that the GRE tunnel is up and that routing information is correctly exchanged over the tunnel.

<br>

### **2 - HQ-SRV-INFRA-01 to Remote Site Gateway (Inter-VLAN Routing test)**

This test verifies end-to-end routed connectivity from **HQ-SRV-INFRA-01** to the Branch-01 local gateways **through the GRE tunnel.**

**HQ-SRV-INFRA-01 $\rightarrow$ BR1-EDGE-R4 (Sales Gateway)**

Plaintext

```
ping 10.60.70.1
```
![](images/Pasted%20image%2020260111010615.png)

**This confirms that internal HQ traffic is correctly routed through the core, edge, and GRE tunnel. Successful replies from 10.60.70.1 prove that the Branch router can receive traffic from the HQ server and send it back through the tunnel. This is a prepare for a functional DHCP Relay service.**

<br>

### **BR1-EDGE-R4: OSPF Verification over GRE Tunnel**


**This verification confirms that the Branch-01 edge router forms a valid OSPF neighbor relationship with the headquarters edge router over the GRE tunnel.** It also verifies that internal headquarters networks and loopback addresses are successfully learned via OSPF through the tunnel.

##### **Verification Commands**

```plaintext
show ip ospf neighbor
show ip route ospf
show ip ospf interface brief
```
![](images/Pasted%20image%2020260111021140.png)
### **Results**

- The headquarters edge router is visible as a FULL OSPF neighbor on the Tunnel0 interface.
    
- OSPF-learned routes from the headquarters site are present in the routing table and are learned via the GRE tunnel.
    
- The loopback interface on the branch edge router is active in OSPF area 1 and correctly advertised.
    
- The GRE tunnel interface operates as a point-to-point OSPF interface with a stable neighbor relationship.


<br>

### **3 - External Security & Isolation Test**

This test verifies that the internal network is isolated from the public internet. It proves that the GRE tunnel is a private path and that ISP/Internet devices cannot reach internal branch resources.

**ISP-BR1 / Internet-Cloud $\rightarrow$ BR1-EDGE-R4 (Ofiice-01 Gateway)**

```
ping 10.60.50.1
```
![](images/Pasted%20image%2020260111011447.png)

**Internet-Cloud $\rightarrow$ BR1-EDGE-R4  (Ofiice-01 Gateway)**

```
ping 10.60.50.1
```
![](images/Pasted%20image%2020260111011409.png)


Pings from the Branch ISP and and the Internet Cloud to any internal VLAN gateway (10.60.50.1) fail with "Unreachable" or timeout. This is the expected and correct behavior in a real-world enterprise network for reason:

* *Tunnel Security: Traffic is encapsulated. The ISP only sees the GRE headers between public IPs, but it has no visibility or route into the private subnets.*


### **Server to Branch Edge Loopback Connectivity**

HQ-SRV-INFRA-01 → BR1-EDGE-R4 (Loopback0 – 10.60.255.1)

```plaintext
ping 10.60.255.1
```
![](images/Pasted%20image%2020260111024446.png)


### **Results**

GRE tunnel connectivity between headquarters and branch is fully operational. Routing over the tunnel functions as expected, enabling inter-site communication. The branch edge router loopback is reachable from the HQ infrastructure server. This confirms end-to-end connectivity over OSPF and the GRE tunnel. The network is now prepared for DHCP relay and further service integration in the following chapters.

<br>

## **6.7 Conclusion**

This chapter completed the site-to-site connectivity between Headquarters and Branch using a GRE tunnel over the simulated Internet. The tunnel provides a secure logical path for inter-site traffic, enables dynamic routing exchange via OSPF, and keeps internal addressing isolated from the ISP and Internet-Cloud. With GRE connectivity verified and routing in place, the network is now prepared for centralized services. The next chapter focuses on configuring DHCP, HTTP, and DNS services on the infrastructure server to support both sites.

<br>

---


<br>

**Next Chapter:** [Core Server Services](07-core-server-services.md)
