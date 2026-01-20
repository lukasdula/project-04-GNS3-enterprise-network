
<br>

# **8 - NAT PAT Configuration**


<br>


## **8.1 NAT/PAT Introduction**

This chapter explains how NAT with PAT is used to provide Internet access for internal VLANs in both Headquarters and Branch sites. Private IP addresses used inside the network cannot be routed on the public Internet, so address translation is required at the edge routers connected to the ISP.

NAT/PAT is applied only on edge routers, where selected user and server VLANs are allowed to access external networks. Management, printer, and inter-site traffic is excluded from translation to keep internal communication secure and routing over the GRE tunnel clean and predictable.


<br>

## **8.2 Topology Diagram**

![](images/Pasted%20image%2020260114015831.png)


<br>


## **8.3 NAT/PAT Objectives**

- Configure NAT with PAT on headquarters and branch edge routers to allow selected internal VLANs to access external networks through the ISP.
    
- Verify correct NAT/PAT operation by generating traffic from both sites and validating address translation using connectivity tests and NAT statistics.


<br>

## **8.4 NAT-PAT Configuration**

<br>

### **Headquarters - NAT ACL and PAT Setup (HQ-EDGE-R3)**

Headquarters outbound traffic is translated using PAT to allow internal VLAN networks to access external resources through the ISP connection. Inter-site traffic between Headquarters and Branch continues to use the GRE tunnel and is excluded from NAT.

**Key logic used in this configuration:**

- NAT is applied on HQ-EDGE-R3 only.
    
- Inside direction is the traffic coming from HQ core routers (HQ-CORE-R1 and HQ-CORE-R2).
    
- Outside direction is the WAN link toward HQ-ISP.
    
- Only selected HQ user and server VLANs are allowed to use NAT/PAT.
    
- Management VLAN 99 is not translated and does not use Internet access.
    
- Inter-site traffic (HQ to Branch) is excluded from NAT to keep GRE-based routing clean.
    

#### **HQ-EDGE-R3 - NAT ACL and Interface Roles**

**Inside interfaces:**

- GigabitEthernet0/1 (uplink to HQ-CORE-R1)
    
- GigabitEthernet0/2 (uplink to HQ-CORE-R2)
    

**Outside interface:**

- GigabitEthernet0/5 (WAN uplink to HQ-ISP)
    

**Allowed HQ networks for NAT/PAT:**

- VLAN 10 - 10.50.10.0/26
    
- VLAN 20 - 10.50.20.0/26
    
- VLAN 30 - 10.50.30.0/24
    
- VLAN 40 - 10.50.40.0/24
    

**Excluded from NAT/PAT:**

- Inter-site traffic to Branch networks (10.60.0.0/16)
    
- VLAN 99 - 10.50.99.0/26
    

#### **HQ-EDGE-R3 - Configuration**

```prikaz
enable
configure terminal
ip access-list extended NAT-HQ
deny ip 10.50.0.0 0.0.255.255 10.60.0.0 0.0.255.255
permit ip 10.50.10.0 0.0.0.63 any
permit ip 10.50.20.0 0.0.0.63 any
permit ip 10.50.30.0 0.0.0.255 any
permit ip 10.50.40.0 0.0.0.255 any
exit
interface GigabitEthernet0/1
ip nat inside
exit
interface GigabitEthernet0/2
ip nat inside
exit
interface GigabitEthernet0/5
ip nat outside
exit
ip nat inside source list NAT-HQ interface GigabitEthernet0/5 overload
end
write
```
![](images/Pasted%20image%2020260114003029.png)

### **Results**

- PAT is enabled on HQ-EDGE-R3 for selected HQ VLAN networks.
    
- HQ internal addresses are translated to the HQ-EDGE-R3 WAN interface address when accessing external networks.
    
- Inter-site traffic to Branch networks is excluded from NAT and continues to use GRE-based routing.
    
- Management VLAN 99 is not included in NAT/PAT and does not use Internet access.

<br>

### **!!! NAT-PAT Troubleshooting Note - HQ Default Route Missing !!!**

NAT/PAT testing from HQ VLANs fails because HQ core routers do not have a default route toward the HQ edge router. The issue is resolved by advertising the default route from HQ-EDGE-R3 into OSPF using `default-information originate`. A full step-by-step this issue is documented in the Troubleshooting chapter.

#### **HQ-EDGE-R3 - Fix**

```prikaz
enable
configure terminal
router ospf 1
default-information originate
end
write
```
![](images/Pasted%20image%2020260114011018.png)
### **Results**

- HQ core routers learn the default route via OSPF.
    
- HQ VLAN traffic reaches the Internet through HQ-EDGE-R3.
    
- NAT/PAT operates correctly on the HQ edge router.



<br>


### **Branch-01 - NAT ACL and PAT Setup (BR1-EDGE-R4)**

Branch-01 outbound traffic is translated using PAT to allow internal VLAN networks to access external resources through the ISP connection. Inter-site traffic between Branch-01 and Headquarters continues to use the GRE tunnel and is excluded from NAT.

**Key logic used in this configuration:**

- NAT is applied on the Branch edge router only.
    
- Inside direction includes selected Branch VLAN subinterfaces configured on BR1-EDGE-R4.
    
- Outside direction is the WAN link toward BR1-ISP.
    
- Only user VLANs that require Internet access are allowed to use NAT/PAT.
    
- Printer and management VLANs are excluded from NAT/PAT.
    
- Inter-site traffic to Headquarters networks is excluded from NAT to preserve GRE-based routing.
    

#### **BR1-EDGE-R4 - NAT ACL and Interface Roles**

**Inside interfaces:**

- GigabitEthernet0/1.50 (VLAN 50 - Office-01)
    
- GigabitEthernet0/1.70 (VLAN 70 - Sales)
    
- GigabitEthernet0/2.60 (VLAN 60 - Office-02)
    

**Outside interface:**

- GigabitEthernet0/5 (WAN uplink to BR1-ISP)
    

**Allowed Branch networks for NAT/PAT:**

- VLAN 50 - 10.60.50.0/24
    
- VLAN 60 - 10.60.60.0/24
    
- VLAN 70 - 10.60.70.0/24
    

**Excluded from NAT/PAT:**

- Inter-site traffic to Headquarters networks (10.50.0.0/16)
    
- VLAN 80 - Printer
    
- VLAN 99 - Management
    

#### **BR1-EDGE-R4 - Configuration**

```prikaz
enable
configure terminal
ip access-list extended NAT-BR1
deny ip 10.60.0.0 0.0.255.255 10.50.0.0 0.0.255.255
permit ip 10.60.50.0 0.0.0.255 any
permit ip 10.60.60.0 0.0.0.255 any
permit ip 10.60.70.0 0.0.0.255 any
exit
interface GigabitEthernet0/1.50
ip nat inside
exit
interface GigabitEthernet0/1.70
ip nat inside
exit
interface GigabitEthernet0/2.60
ip nat inside
exit
interface GigabitEthernet0/5
ip nat outside
exit
ip nat inside source list NAT-BR1 interface GigabitEthernet0/5 overload
end
write
```
![](images/Pasted%20image%2020260114003106.png)

### **Results**

- PAT is enabled on BR1-EDGE-R4 for selected Branch user VLANs.
    
- Branch internal addresses are translated to the BR1-EDGE-R4 WAN interface address when accessing external networks.
    
- Inter-site traffic to Headquarters networks is excluded from NAT and continues to use GRE-based routing.
    
- Printer and management VLANs do not use Internet access.

<br>

## **8.5 NAT/PAT Final Verification - HQ and Branch**

This last section verifies correct NAT/PAT operation on both sites by generating real traffic and validating address translation directly on the edge routers.

<br>

### **1) Headquarters - Connectivity and NAT Verification**

Traffic is generated from the headquarters infrastructure server toward the ISP-facing WAN interface.

**Traffic flow:** HQ-SRV-INFRA-01 -> HQ-EDGE-R3 -> HQ-ISP

```plaintext
ping 37.48.0.1
```
![](images/Pasted%20image%2020260114013131.png)

#### HQ-EDGE-R3 - NAT Verification

```plaintext
show ip nat translations
show ip nat statistics
```
![](images/Pasted%20image%2020260114013211.png)

The output confirms that internal HQ addresses are translated to the public ISP-facing address using PAT overload.

<br>

### **2) Branch-01 - Connectivity and NAT Verification**

Traffic is generated from a branch VLAN client toward the branch ISP WAN interface.

**Traffic flow:** BR1-SALES-01 -> BR1-EDGE-R4 -> BR1-ISP

```plaintext
ping 89.203.0.1
```
![](images/Pasted%20image%2020260114013254.png)

#### **BR1-EDGE-R4 - NAT Verification**

```plaintext
show ip nat translations
show ip nat statistics
```
![](images/Pasted%20image%2020260114013310.png)

The output confirms correct address translation on the branch edge router.

<br>

### **NAT/PAT Address Translation Overview**

The table below illustrates how an internal branch device is represented externally using PAT.

| Term           | Example Value | Meaning                              | Explanation                                                  |
| -------------- | ------------- | ------------------------------------ | ------------------------------------------------------------ |
| Inside Local   | 10.60.70.100  | Internal private address             | IPv4 address of a device inside the Branch Sales VLAN.       |
| Inside Global  | 89.203.0.2    | Public address for inside            | ISP-facing WAN address of BR1-EDGE-R4 used for PAT overload. |
| Outside Local  | 89.203.0.1    | External host as seen from inside    | ISP router address as viewed from the internal network.      |
| Outside Global | 89.203.0.1    | Real public address of external host | Globally reachable IPv4 address of the ISP router.           |

### Results

NAT with PAT operates correctly on both headquarters and branch edge routers. Internal VLAN traffic is successfully translated to public addresses and forwarded toward the ISP while private addressing remains hidden. End-to-end connectivity and translation behavior are verified on both sites.

<br>

## **8.6 Conclusion**

NAT with PAT is successfully configured on both headquarters and branch edge routers. Selected internal VLANs can access external networks using the ISP connection while private addressing is preserved.

Inter-site traffic over the GRE tunnel and management networks are not changed by NAT, providing stable routing and clear separation between internal communication and Internet access. The network is now fully prepared for the next security chapter, which focuses on access control and testing remote administrator access from a simulated SOHO home environment into the internal enterprise network.



<br>

---

<br>

**Next Chapter:**
