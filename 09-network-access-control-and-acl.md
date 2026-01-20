

<br>



# **9 - Network Access Control and Acl**

<br>

## **9.1 Introduction**

Remote administrative access in this chapter is demonstrated first using PuTTY (SSH) from the Remote-Admin network. This confirms that key management targets are reachable over VLAN 99 (management) and that routers can be accessed for configuration and troubleshooting.

Access Control Lists (ACLs) are then introduced as a simple security baseline for both sites (HQ and Branch). Infrastructure, Monitoring, and Management networks stay trusted, while user VLANs are restricted to reduce unwanted inter-VLAN access.

The verification steps confirm the intended result: required services (DHCP, DNS, HTTP, syslog) remain available, monitoring stays functional, and blocked paths are proven blocked without breaking core connectivity.

<br>

## **9.2 Topology diagram**

![](images/Pasted%20image%2020260114081631.png)


<br>

## **9.5 Objectives**

1. Verify remote administration access using PuTTY to ensure that network devices can be securely managed from the administrator workstation.
    
2. Observe SSH traffic in Wireshark to demonstrate encrypted remote management and confirm secure communication over the management network.
    
3. Implement Access Control Lists (ACLs) to control inter-VLAN and inter-site traffic while allowing required infrastructure and management services.
    
4. Apply the ACL policy consistently on HQ and Branch routing devices to apply the same security rules across both sites.
    
5. Validate the ACL function through verification tests, confirming that permitted traffic is allowed and unauthorized traffic is correctly blocked.





<br>


## **9.4 Remote Administrative Access (PUTTY-SSH)**

This section introduces remote administrative access to the enterprise network from an external SOHO environment. SSH is used to manage selected network devices over a public network.

The purpose of this section is to demonstrate a simple remote administration workflow and to verify that network devices are accessible from outside the enterprise network.


- SSH configuration on an BR1-EDGE-R4
    
- SSH configuration on a management BR1-DIST-SW-01 (VLAN 99)
    
- Basic preparation for remote administrative access

<br>

**Remote administrative access path from REMOTE-ADMIN environment to Branch management infrastructure.**

![](images/Pasted%20image%2020260114040316.png)


<br>


>Note: In real enterprise networks, remote administrative access is usually provided through a VPN. VPN configuration is not part of this project and will be covered in a separate security-focused project.
 This chapter uses direct SSH access with PuTTY to demonstrate basic remote device management. More advanced security mechanisms will be addressed later.

<br>

### **SSH Configuration – BR1-EDGE-R4

This first configuration enables SSH access on the headquarters edge router. Local user authentication is used for both console and remote access.

#### Example – BR1-EDGE-R4 SSH Configuration

```plaintext
enable
configure terminal
enable secret corp
username admin privilege 15 secret corp
ip domain-name corp.local
crypto key generate rsa 
2048
ip ssh version 2
line console 0
login local
exec-timeout 10 0
exit
line vty 0 15
login local
transport input ssh
exec-timeout 10 0
exit
end
write
```
![](images/Pasted%20image%2020260114035514.png)

> Note: The username and password used here are intentionally simple for learning purposes. In production environments, strong passwords and additional advanced security mechanisms are required.


<br>

### **SSH Configuration – BR1-DIST-SW-01 (VLAN 99)

This configuration enables SSH access on a branch distribution switch using the management VLAN.

#### Example – BR1-DIST-SW-01 SSH Configuration

```plaintext
enable
configure terminal
enable secret corp
username admin privilege 15 secret corp
ip domain-name corp.local
crypto key generate rsa 
2048
ip ssh version 2
line console 0
login local
exec-timeout 10 0
exit
line vty 0 15
login local
transport input ssh
exec-timeout 10 0
exit
end
write
```
![](images/Pasted%20image%2020260114032604.png)

<br>

### Results

SSH access is enabled on the branch edge router and on the branch management switch. Local user authentication is enabled on console and VTY lines. The devices are prepared for remote administrative access from an external administrator workstation.

<br>


## Remote Administrative Access Verification

This section verifies remote administrative access to the enterprise network from an external administrator workstation using SSH.

The connection path verified in this section:

- Remote-Admin (SOHO) → BR1-EDGE-R4 (public WAN)
    
- BR1-EDGE-R4 → BR1-DIST-SW-01 (VLAN 99)
    

This confirms that remote management of network infrastructure is possible and that the management VLAN is reachable through routed connectivity. An ACL exception for this access path will be applied in the following section.

<br>

## Remote-Admin to BR1-EDGE-R4 (SSH)

This step verifies SSH connectivity from the external administrator workstation to the branch edge router using the public WAN address.

### Connection Parameters

- Source: Remote-Admin
    
- Destination: BR1-EDGE-R4
    
- Destination IP: 89.203.0.2
    
- Protocol: SSH
    
- Port: 22
    

### Example – PuTTY Session

```plaintext
PuTTY SSH connection to 89.203.0.2 on port 22
```
![](images/Pasted%20image%2020260114040551.png)

<br>

### BR1-EDGE-R4 to BR1-DIST-SW-01 (SSH)

This step verifies SSH access from the branch edge router to the branch distribution switch using the management VLAN.

#### **SSH from BR1-EDGE-R4**

```plaintext
ssh -l admin 10.60.99.50
```
![](images/Pasted%20image%2020260114040720.png)


### Verification

The following commands verify active SSH sessions on the edge router and the branch switch.

```plaintext
show ssh
show users
```
![](images/Pasted%20image%2020260114040809.png)


### Results

Remote SSH access from the external administrator workstation to the branch edge router is successful. SSH connectivity from the edge router to the branch management switch using VLAN 99 is also operational. Remote administrative management of the enterprise network is confirmed.

<br>

## **9.5 SSH Traffic Monitoring (Wireshark)**

This section demonstrates monitoring of SSH traffic between the branch edge router and the branch distribution switch using Wireshark. The goal is to verify that SSH communication is established correctly and that the traffic is encrypted.

The capture is performed on the link between **BR1-EDGE-R4** and **BR1-DIST-SW-01**, where management traffic to VLAN 99 is routed.

### **Capture Location**

- Device: BR1-EDGE-R4 ↔ BR1-DIST-SW-01
    
- Interface: Gi0/1 (edge) ↔ Gi0/1 (switch)
    
- Traffic type: Management (SSH)
    
![](images/Pasted%20image%2020260114041734.png)


### **Capture Filter**

```plaintext
tcp.port == 22
```
![](images/Pasted%20image%2020260114042048.png)


### **Results**

The capture shows the TCP three-way handshake on port 22, followed by SSH protocol negotiation and key exchange. After the key exchange is completed, all subsequent packets are transmitted as encrypted SSH traffic, confirming secure communication over the management link.


<br>



## **9.6 Network Access Control and ACL**


This next chapter defines a simple and practical access control baseline for the enterprise network. The focus is on controlling traffic between VLANs while keeping core services, monitoring, and management fully functional across both sites (Headquarters and Branch).

The design balances security and operability. User VLANs are restricted to limit **unnecessary** movement between VLANs, while infrastructure, monitoring, and management networks are trusted to ensure stable network operation, visibility, and effective troubleshooting.

### **Policy objectives**

The ACL policy follows the same logical order on both sites and is applied inbound on VLAN gateway interfaces.

- **Trusted Networks**: Infrastructure (VLAN 10), Monitoring (VLAN 20), and Management (VLAN 99) are treated as trusted zones with full network reachability.
    
- ICMP is allowed for monitoring and troubleshooting from Management and Infra/MON servers networks. Inter-VLAN communication between standard users is prohibited (Drop) and to vlan99. Users can only ping their default gateway.
    
- **Service Access for User VLANs**: User VLANs are limited to required services only (DHCP, DNS, HTTP, and centralized logging).
    
- **Inter-site Services**: Shared resources, such as the branch printer, are explicitly permitted.
    
- **Default Deny**: All other inter-VLAN traffic is denied to enforce basic segmentation.
    

<br>

### **HQ Configuration (HQ-CORE-R1 and HQ-CORE-R2)**

The same ACL is applied identically on both HQ core routers to ensure consistent behavior during VRRP failover events.

```plaintext
configure terminal
ip access-list extended HQ-ENTERPRISE-ACL
permit udp any any eq 67
permit udp any any eq 68
deny icmp 10.50.30.0 0.0.0.255 10.50.99.0 0.0.0.63 echo
deny icmp 10.50.40.0 0.0.0.255 10.50.99.0 0.0.0.63 echo
deny icmp 10.50.30.0 0.0.0.255 10.60.99.0 0.0.0.63 echo
deny icmp 10.50.40.0 0.0.0.255 10.60.99.0 0.0.0.63 echo
permit icmp any 10.50.10.0 0.0.0.63 echo-reply
permit icmp any 10.50.20.0 0.0.0.63 echo-reply
permit icmp any 10.50.99.0 0.0.0.63 echo-reply
permit icmp any 10.50.0.0 0.0.255.255 echo
permit icmp any 10.60.0.0 0.0.255.255 echo
permit icmp any any echo
permit icmp 10.50.10.0 0.0.0.63 any echo
permit icmp 10.50.20.0 0.0.0.63 any echo
permit icmp 10.50.99.0 0.0.0.63 any echo
permit ip 10.50.10.0 0.0.0.63 any
permit ip 10.50.20.0 0.0.0.63 any
permit ip 10.50.99.0 0.0.0.63 any
permit udp any 10.50.10.0 0.0.0.63 eq 53
permit tcp any 10.50.10.0 0.0.0.63 eq 53
permit tcp any 10.50.10.0 0.0.0.63 eq 80
permit udp any 10.50.10.0 0.0.0.63 eq 514
permit tcp any 10.60.80.0 0.0.0.31 eq 9100
deny ip any any
exit
interface GigabitEthernet0/0.10
ip access-group HQ-ENTERPRISE-ACL in
interface GigabitEthernet0/0.20
ip access-group HQ-ENTERPRISE-ACL in
interface GigabitEthernet0/0.30
ip access-group HQ-ENTERPRISE-ACL in
interface GigabitEthernet0/0.40
ip access-group HQ-ENTERPRISE-ACL in
interface GigabitEthernet0/0.99
ip access-group HQ-ENTERPRISE-ACL in
end
write memory

```
![](images/Pasted%20image%2020260119001140.png)

<br>

### **ACL Port and Service Analysis**

This table explains the logic and purpose of each permitted service in the ACL:

| **Order** | **Protocol / Port** | **Service**      | **Meaning and Logic**                                                                                               |
| --------- | ------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------- |
| **1.**    | **UDP 67, 68**      | **DHCP**         | **Essential.** Allows PCs to get IP addresses from HQ-SRV-INFRA-01 using `any` source.                              |
| **2.**    | **ICMP**            | **Ping / Trace** | **Restricted.** Allows monitoring from Zabbix (VLAN 20) and pings to Default Gateway. Blocks inter-VLAN user pings. |
| **3.**    | **IP (trusted)**    | **Full Access**  | **Management.** Full access for Trusted VLANs 10, 20, 99 (Zabbix, Servers, Admin).                                  |
| **4.**    | **UDP/TCP 53**      | **DNS**          | Allows hostname resolution via dnsmasq (HQ-SRV-INFRA-01).                                                           |
| **5.**    | **TCP 80**          | **HTTP**         | Allows access to the internal web portal (Apache).                                                                  |
| **6.**    | **UDP 514**         | **rsyslog**      | Allows devices to send system logs to the central server.                                                           |
| **7.**    | **TCP 9100 / ICMP** | **Printing**     | Allows printing and availability checks (ping) for the Branch printer.                                              |
| **8.**    | **IP (any any)**    | **Deny All**     | **Security Baseline.** Blocks all unauthorized inter-VLAN and external traffic.                                     |


### **Results**

- User VLANs are isolated from each other, **reducing the spreading** of traffic between departments.
    
- Infrastructure, monitoring, and management networks retain full visibility and control.
    
- Zabbix and syslog can communicate reliably with all network devices.
    
- Core services such as DHCP, DNS, and HTTP remain available to users.
    
- Identical ACLs on both core routers ensure stable operation during failover events.
    

<br>

## **Branch ACL Configuration (BR1)**

Remote access, monitoring, and core services must keep working across the HQ <-> Branch connection, while normal user VLANs stay separated.

This Branch policy follows the same baseline idea as HQ:

- Trusted networks (Monitoring and Management) can reach devices.
    
- ICMP is allowed for monitoring and troubleshooting from Management and Infra networks. Inter-VLAN communication between standard users is prohibited (Drop). Users can only ping their default gateway.
    
- User VLANs can use only essential services.
    
- Everything else is blocked.
    

>Notes: This is a basic ACL baseline for the project. A stricter, port-by-port security policy (and remote admin over VPN) is handled in a separate security-focused project.


<br>

### **Policy objectives**

- HQ Monitoring VLAN 20 can monitor Branch devices (future Zabbix chapter).
    
- For security policy reasons, Branch Management (VLAN 99) remains accessible for local administration, but access from user VLANs is restricted.
    
- Branch user VLANs can reach HQ Infra services (DHCP, DNS, HTTP, syslog).
    
- Branch Printer VLAN stays reachable for printing.
    
- Inter-VLAN user traffic is blocked by default.
    

<br>

## Configuration (BR1-EDGE-R4)

```prikaz
configure terminal
no ip access-list extended BRANCH-ENTERPRISE-ACL
permit udp any any eq 67
permit udp any any eq 68
deny icmp 10.60.50.0 0.0.0.255 10.60.99.0 0.0.0.63 echo
deny icmp 10.60.60.0 0.0.0.255 10.60.99.0 0.0.0.63 echo
deny icmp 10.60.70.0 0.0.0.255 10.60.99.0 0.0.0.63 echo
deny icmp 10.60.50.0 0.0.0.255 10.50.99.0 0.0.0.63 echo
deny icmp 10.60.60.0 0.0.0.255 10.50.99.0 0.0.0.63 echo
deny icmp 10.60.70.0 0.0.0.255 10.50.99.0 0.0.0.63 echo
permit icmp any 10.50.10.0 0.0.0.63 echo-reply
permit icmp any 10.50.20.0 0.0.0.63 echo-reply
permit icmp any 10.50.99.0 0.0.0.63 echo-reply
permit icmp any 10.60.99.0 0.0.0.63 echo-reply
permit icmp any 10.60.0.0 0.0.255.255 echo
permit icmp any 10.50.0.0 0.0.255.255 echo
permit icmp any any echo
permit icmp 10.50.10.0 0.0.0.63 any echo
permit icmp 10.50.20.0 0.0.0.63 any echo
permit icmp 10.50.99.0 0.0.0.63 any echo
permit icmp 10.60.99.0 0.0.0.63 any echo
permit ip 10.50.20.0 0.0.0.63 any
permit ip 10.50.99.0 0.0.0.63 any
permit ip 10.60.99.0 0.0.0.63 any
permit udp any 10.50.10.0 0.0.0.63 eq 53
permit tcp any 10.50.10.0 0.0.0.63 eq 53
permit tcp any 10.50.10.0 0.0.0.63 eq 80
permit udp any 10.50.10.0 0.0.0.63 eq 514
permit tcp any 10.60.80.0 0.0.0.31 eq 9100
deny ip any any
exit
interface GigabitEthernet0/1.50
ip access-group BRANCH-ENTERPRISE-ACL in
interface GigabitEthernet0/2.60
ip access-group BRANCH-ENTERPRISE-ACL in
interface GigabitEthernet0/1.70
ip access-group BRANCH-ENTERPRISE-ACL in
interface GigabitEthernet0/1.99
ip access-group BRANCH-ENTERPRISE-ACL in
end
write memory

```
![](images/Pasted%20image%2020260119001210.png)

<br>

### **ACL Port and Service Analysis**

|**Order**|**Protocol / Port**|**Service**|**Meaning and Logic**|
|---|---|---|---|
|**1.**|**UDP 67, 68**|**DHCP**|**Essential.** Allows PCs to get IP addresses from HQ-SRV-INFRA-01 using `any` source.|
|**2.**|**ICMP**|**Ping / Trace**|**Restricted.** Allows monitoring from Zabbix (VLAN 20) and pings to Default Gateway. Blocks inter-VLAN user pings.|
|**3.**|**IP (trusted)**|**Full Access**|**Management.** Full access for Trusted VLANs 10, 20, 99 (Zabbix, Servers, Admin).|
|**4.**|**UDP/TCP 53**|**DNS**|Allows hostname resolution via dnsmasq (HQ-SRV-INFRA-01).|
|**5.**|**TCP 80**|**HTTP**|Allows access to the internal web portal (Apache).|
|**6.**|**UDP 514**|**rsyslog**|Allows devices to send system logs to the central server.|
|**7.**|**TCP 9100 / ICMP**|**Printing**|Allows printing and availability checks (ping) for the Branch printer.|
|**8.**|**IP (any any)**|**Deny All**|**Security Baseline.** Blocks all unauthorized inter-VLAN and external traffic.|

<br>

## Results

- HQ Monitoring (VLAN 20) keeps full visibility to Branch devices for the next Zabbix chapter.
    
- Branch Management (VLAN 99) remains usable for local administration; however, due to management security policy, user VLANs cannot reach VLAN 99 via ping.
    
- Branch user VLANs can still use DHCP, DNS, HTTP, and syslog via HQ Infra.
    
- Printer access stays available, while other user-to-user VLAN traffic is blocked by default.

<br>



## 9.7 ACL Verification and Validation

This last section verifies that the implemented ACL policies behave as intended across both Headquarters (HQ) and Branch sites. 

The goal is to confirm correct traffic isolation between user VLANs, continued availability of infrastructure services, and uninterrupted inter-site connectivity for monitoring, management, and routing.

The verification focuses on real traffic behavior using ICMP tests and basic ACL inspection commands after policy deployment.

<br>




### **1) Inter-VLAN Isolation Tests**

<br>

#### HQ – User VLAN Isolation

- **VLAN 30 (Office) → VLAN 40 (Finance)**
  
 ```prikaz
   ping 10.50.40.101
 ```
![](images/Pasted%20image%2020260114213639.png)


 Expected result: **Fail** (user VLANs are isolated).
 

#### Branch – User VLAN Isolation

- **VLAN 50 (Office-01) → VLAN 60 (Office-02)**

 ```prikaz
ping 10.60.60.100
 ```
![](images/Pasted%20image%2020260114213849.png)


Expected result: **Fail**.

- **VLAN 60 (Office-02) → VLAN 70 (Sales)**
 
```prikaz
ping 10.60.70.100
```



Expected result: **Fail**.


<br>

### **2) Inter-Site User VLAN Tests**

- **HQ VLAN 30 → Branch VLAN 50 (Office-01)**
    
```prikaz
 ping 10.60.50.100
```
![](images/Pasted%20image%2020260114214158.png)


Expected result: **Fail** (no direct user-to-user access across sites).

- **Branch VLAN 60 (Office-02) → HQ VLAN 40 Finance)**

```prikaz
ping 10.50.40.100
```
![](images/Pasted%20image%2020260114214345.png)


Expected result: **Fail**.


<br>

### **3) Infrastructure and Server Access Tests**

#### User VLANs to HQ Infrastructure Server

- **HQ VLAN 40 → HQ-INFRA Server (VLAN 10)**
```prikaz
ping 10.50.10.10
```
![](images/Pasted%20image%2020260114214537.png)

Expected result: **Success**.
 
* Branch VLAN 50 → HQ-INFRA Server (VLAN 10)
```prikaz
ping 10.50.10.10
```
![](images/Pasted%20image%2020260114214611.png)


Expected result: **Success**.


#### Server-to-Server Communication

- **HQ-INFRA Server → HQ-MON Server**
- HQ-INFRA Server → BR-Office-01

```prikaz
ping 10.50.20.10
```
![](images/Pasted%20image%2020260114214847.png)


Expected result: **Success**.

<br>


### **4) Printer Access Tests (Branch VLAN 80)**

- **HQ VLAN 40 → Branch Printer (VLAN 80)**

```prikaz
ping 10.60.80.10
```
![](images/Pasted%20image%2020260114214958.png)


Expected result: **Success**.

- **Branch VLAN 70 → Branch Printer (VLAN 80)**
 
```prikaz
ping 10.60.80.10
```
![](images/Pasted%20image%2020260114215025.png)


Expected result: **Success**.


<br>

### 5) Router and Inter-Site Connectivity Tests

- **HQ-EDGE-R3 → BR1-EDGE-R4 (WAN link)**
    
```prikaz
ping 89.203.0.2
```
![](images/Pasted%20image%2020260114215129.png)


Expected result: **Success**.



-HQ-EDGE-R3 **GRE → Tunnel (HQ → Branch)**

```prikaz
ping 10.254.0.2
```
![](images/Pasted%20image%2020260114215205.png)


Expected result: **Success**.
 

<br>

### **6) ACL Inspection and Diagnostics**

#### Branch Edge Router (BR1-EDGE-R4)

```prikaz
show access-lists BRANCH-ENTERPRISE-ACL
```
![](images/Pasted%20image%2020260114215238.png)



#### HQ-CORE-R1

```prikaz
show access-lists HQ-ENTERPRISE-ACL
```
![](images/Pasted%20image%2020260114215340.png)


Expected result: ACL counters increase on permitted rules, with denies matched only for unauthorized inter-VLAN traffic.

<br>


<br>

### Results Summary

- User VLANs are fully isolated within and across sites.
    
- Infrastructure services (DHCP, DNS, HTTP, syslog) remain reachable from all required VLANs.
    
- Printer access is available to authorized VLANs only.
    
- Inter-site routing, WAN connectivity, and GRE tunnel operation remain unaffected by ACL policies.
    
- NAT/PAT functionality is verified as operational, allowing internal hosts to access external networks (WAN) through a single public IP address while maintaining ACL security standards.
    

The verification confirms that the ACL implementation meets the intended security policy while preserving operational functionality across the enterprise network.

<br>

### **9.8 Conclusion**

This chapter starts with remote administration using SSH, which confirms secure access to network devices for management and troubleshooting. Traffic monitoring is then used to observe management communication, followed by the implementation of Access Control Lists to control traffic between VLANs and sites while keeping essential services available.

Verification confirms that allowed communication works as expected and unwanted traffic is blocked. The next chapter focuses on centralized network monitoring using Zabbix, where device availability and network health are monitored from a central server through a web interface.


<br>

---

<br>


**Next Chapter:** [Centralized Monitoring Zabbix](10-centralized-monitoring-zabbix.md)
