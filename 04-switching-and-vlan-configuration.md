
<br>

# **4 - Switching and VLAN Configuration**


<br>

## **4.1 Introduction**

This fourth chapter focuses on the configuration of the Layer 2 switching foundation for the enterprise network. The purpose of this chapter is to create a clear VLAN structure, separate different types of network traffic, and prepare the switching environment before routing is introduced.

The configuration is done step by step. VLANs are created first to represent different network roles such as infrastructure, monitoring, user workstations, printers, and management. Access ports are then configured so that end devices are placed into the correct VLAN based on their function and location.

Next, trunk links are configured between switches and between switches and routers to allow multiple VLANs to pass through the Layer 2 network. IEEE 802.1Q encapsulation is explicitly configured on trunk ports to meet the requirements of the IOSv-L2 switching platform and to ensure consistent VLAN operation across the topology.

Rapid Spanning Tree Protocol (RSTP) is used to protect the Layer 2 topology from switching loops and to ensure stable Layer 2 operation. Root bridge roles are defined at the distribution layer, while PortFast and BPDU Guard are applied on access ports to prevent topology issues caused by end devices. This configuration prepares a stable and loop-free Layer 2 environment for both headquarters and branch locations.

<br>

## **4.2 Topology Diagram**

![](images/Pasted%20image%2020260110044714.png)

<br>


## **4.3 VLAN Overview**

<br>

### **Headquarters (HQ)**

| **VLAN ID** | **VLAN Name**  | **Description / Purpose** | **Assigned Devices**                                     | **Switch Association**                                  |
| ----------: | -------------- | ------------------------- | -------------------------------------------------------- | ------------------------------------------------------- |
|          10 | Infrastructure | Infrastructure servers    | HQ-SRV-INFRA-01                                          | HQ-ACC-SW-01                                            |
|          20 | Monitoring     | Monitoring servers        | HQ-SRV-MON-01                                            | HQ-ACC-SW-01                                            |
|          30 | Office         | Office user workstations  | HQ-OFFICE-01                                             | HQ-ACC-SW-02                                            |
|          40 | Finance        | Finance user workstations | HQ-FINANCE-01                                            | HQ-ACC-SW-02                                            |
|          99 | Management     | Network device management | HQ-DIST-SW-01, HQ-DIST-SW-02, HQ-ACC-SW-01, HQ-ACC-SW-02 | HQ-DIST-SW-01, HQ-DIST-SW-02,HQ-ACC-SW-01, HQ-ACC-SW-02 |

<br>

### **Branch-01 (BR1)**

| **VLAN ID** | **VLAN Name** | **Description / Purpose**                   | **Assigned Devices**           | **Switch Association**         |
| ----------: | ------------- | ------------------------------------------- | ------------------------------ | ------------------------------ |
|          50 | Office-01     | Branch office user workstations (Office-01) | BR1-OFFICE-01                  | BR1-DIST-SW-01                 |
|          60 | Office-02     | Branch office user workstations (Office-02) | BR1-OFFICE-02                  | BR1-DIST-SW-02                 |
|          70 | Sales         | Branch sales workstations                   | BR1-SALES-01                   | BR1-DIST-SW-01                 |
|          80 | Printer       | Branch network printer segment              | BR1-PRINTER-01                 | BR1-DIST-SW-02                 |
|          99 | Management    | Branch network device management            | BR1-DIST-SW-01, BR1-DIST-SW-02 | BR1-DIST-SW-01, BR1-DIST-SW-02 |

<br>


## **4.4 Objectives**

1. Create all required VLANs on every switch according to the network design.
    
2. Assign access ports to the correct VLANs based on connected end devices, including PortFast and BPDU Guard configuration on all access ports.
    
3. Configure trunk ports to carry all required VLANs across the switching layer.
    
4. Configure Rapid Spanning Tree Protocol (RSTP) to control the Layer 2 topology and define root bridge roles.
    
5. Verify VLAN, trunk, access ports and spanning tree configuration using appropriate diagnostic commands.


<br>

## **4.5 VLAN Creation - Headquarters (HQ)**



This section defines VLAN creation for the headquarters site. All VLANs are created on every headquarters switch to ensure consistent Layer 2 operation across trunk links, correct spanning tree operation, and reliable VLAN forwarding throughout the switching infrastructure. Even if a VLAN is not actively used on a specific access switch, it is still created to keep the VLAN configuration the same across the site.

The same VLAN IDs and names are used on both distribution and access switches. This makes the configuration easier to read, helps during verification and troubleshooting, and follows common enterprise switching design rules.

<br>

### **HQ-DIST-SW-01 - VLAN Configuration**

```plaintext
enable
configure terminal
vlan 10
name Infrastructure
vlan 20
name Monitoring
vlan 30
name Office
vlan 40
name Finance
vlan 99
name Management
end
write memory
```
![](images/Pasted%20image%2020260109152607.png)


<br>

### **HQ-DIST-SW-02 - VLAN Configuration**

```plaintext
enable
configure terminal
vlan 10
name Infrastructure
vlan 20
name Monitoring
vlan 30
name Office
vlan 40
name Finance
vlan 99
name Management
end
write memory
```
![](images/Pasted%20image%2020260109152715.png)


<br>

### **HQ-ACC-SW-01 - VLAN Configuration**

```plaintext
enable
configure terminal
vlan 10
name Infrastructure
vlan 20
name Monitoring
vlan 30
name Office
vlan 40
name Finance
vlan 99
name Management
end
write memory
```
![](images/Pasted%20image%2020260109152802.png)


<br>

### **HQ-ACC-SW-02 - VLAN Configuration**

```plaintext
enable
configure terminal
vlan 10
name Infrastructure
vlan 20
name Monitoring
vlan 30
name Office
vlan 40
name Finance
vlan 99
name Management
end
write memory
```
![](images/Pasted%20image%2020260109152839.png)

<br>

### **Results - Headquarters VLAN Creation**

All required VLANs are successfully created on every headquarters switch. The VLAN configuration is consistent across distribution and access layers, ensuring correct VLAN handling on trunk links and stable Layer 2 operation. The headquarters switching environment is now prepared for access port and trunk configuration.



<br>


## **4.6 VLAN Creation - Branch-01 (BR1)**


This next section defines VLAN creation for the branch site. All required VLANs are created on every branch switch to ensure consistent Layer 2 operation across trunk links and predictable switching operation within the branch topology. Even if a VLAN is not directly used on a specific switch, it is still created to keep the VLAN configuration consistent across the site.

The same VLAN IDs and names are applied on both branch distribution switches. This ensures clear verification output, easier troubleshooting, and a consistent switching design aligned with common enterprise practices.

<br>

### **BR1-DIST-SW-01 - VLAN Configuration**

```plaintext
enable
configure terminal
vlan 50
name Office-01
vlan 60
name Office-02
vlan 70
name Sales
vlan 80
name Printer
vlan 99
name Management
end
write memory
```
![](images/Pasted%20image%2020260109152955.png)


<br>

### **BR1-DIST-SW-02 - VLAN Configuration**

```plaintext
enable
configure terminal
vlan 50
name Office-01
vlan 60
name Office-02
vlan 70
name Sales
vlan 80
name Printer
vlan 99
name Management
end
write memory
```
![](images/Pasted%20image%2020260109153042.png)

<br>

### **Results - Branch VLAN Creation**

All required VLANs are successfully created on the branch switches. The VLAN configuration is consistent across the branch distribution layer, ensuring correct VLAN handling on trunk links and stable Layer 2 operation within the branch network.

<br>

## **4.7 Access Port, PortFast and BPDU Guard Configuration**


This section describes access port configuration for the switching layer, including the use of PortFast and BPDU Guard. Access ports are used to connect end devices such as servers, workstations, and printers to the network and are assigned to a single VLAN based on the connected device.

PortFast allows access ports to move immediately into the forwarding state, which reduces startup delays for end devices. BPDU Guard protects the Layer 2 topology by automatically disabling an access port if a BPDU is received, preventing accidental loops caused by incorrect device connections. Together, these features improve stability and safety in the enterprise switching design.

<br>

### **Headquarters (HQ)**

#### HQ-ACC-SW-01 - Access Port Configuration

    
- **Gi0/0**
    
    - Device: HQ-SRV-INFRA-01
        
    - VLAN: 10 (HQ-Infrastructure)
        
    - Role: Access port (services - DHCP/ DNS / HTTP)
        
- **Gi0/2**
    
    - Device: HQ-SRV-MON-01
        
    - VLAN: 20 (HQ-Monitoring)
        
    - Role: Access port (monitoring server-Zabbix)

<br>

```plaintext
enable
configure terminal
interface GigabitEthernet0/0
switchport mode access
switchport access vlan 10
spanning-tree portfast
spanning-tree bpduguard enable
no shutdown
exit
interface GigabitEthernet0/2
switchport mode access
switchport access vlan 20
spanning-tree portfast
spanning-tree bpduguard enable
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260109153212.png)

>**Notes.:** The output confirms that the interfaces are correctly configured as access ports with assigned VLANs and PortFast enabled. The warning messages are informational and indicate that PortFast and BPDU Guard are active only on non-trunk ports, which matches the intended access port design.



<br>

#### HQ-ACC-SW-02 - Access Port Configuration


- **Gi0/0**
    
    - Device: HQ-OFFICE-01
        
    - VLAN: 30 (HQ-Office)
        
    - Role: Access port (office workstation)
        
- **Gi0/1**
    
    - Device: HQ-FINANCE-01
        
    - VLAN: 40 (HQ-Finance)
        
    - Role: Access port (finance workstation)

<br>

```plaintext
enable
configure terminal
interface GigabitEthernet0/0
switchport mode access
switchport access vlan 30
spanning-tree portfast
spanning-tree bpduguard enable
no shutdown
exit
interface GigabitEthernet0/1
switchport mode access
switchport access vlan 40
spanning-tree portfast
spanning-tree bpduguard enable
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260109153345.png)

<br>

---

<br>

### **Branch-01 (BR1)**

<br>

#### BR1-DIST-SW-01 - Access Port Configuration


- **Gi0/2**
    
    - Connected device: BR1-OFFICE-01
        
    - VLAN: 50 (Office-01)
        
    - Port role: (office workstation)
        
- **Gi0/3**
    
    - Connected device: BR1-SALES-01
        
    - VLAN: 70 (Sales)
        
    - Port role: (finance workstation)
        

<br>

```plaintext
enable
configure terminal
interface GigabitEthernet0/2
switchport mode access
switchport access vlan 50
spanning-tree portfast
spanning-tree bpduguard enable
no shutdown
exit
interface GigabitEthernet0/3
switchport mode access
switchport access vlan 70
spanning-tree portfast
spanning-tree bpduguard enable
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260109153433.png)

<br>

#### BR1-DIST-SW-02 - Access Port Configuration


- **Gi0/1**
    
    - Connected device: BR1-OFFICE-02
        
    - VLAN: 60 (Office-02)
        
    - Port role: (office workstation)
        
- **Gi0/3**
    
    - Connected device: BR1-PRINTER-01
        
    - VLAN: 80 (Printer)

<br>

```plaintext
enable
configure terminal
interface GigabitEthernet0/1
switchport mode access
switchport access vlan 60
spanning-tree portfast
spanning-tree bpduguard enable
no shutdown
exit
interface GigabitEthernet0/3
switchport mode access
switchport access vlan 80
spanning-tree portfast
spanning-tree bpduguard enable
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260109153616.png)


<br>

### **Results - Access Port Configuration**

All access ports are correctly assigned to their respective VLANs based on connected end devices. PortFast and BPDU Guard are enabled on all access ports, ensuring fast port activation and protection against accidental Layer 2 loops. The switching environment is now ready for trunk configuration.


<br>

## **4.8 Configure Trunk Ports - Headquarters (HQ)**


This section configures trunk ports within the headquarters switching layer. Trunk links are used between switches to carry multiple VLANs across the network and to support efficient and stable Layer 2 connectivity between distribution and access layers. Allowing all required VLANs on trunk links ensures consistent VLAN forwarding and predictable traffic behavior across the switching infrastructure.

The switching devices in this project use an IOSv-L2 image, which requires explicit configuration of 802.1Q encapsulation on trunk ports. For this reason, trunk encapsulation is defined manually on all trunk interfaces.

<br>

### **HQ-DIST-SW-01 - Trunk Ports**

Trunk ports on HQ-DIST-SW-01:

- Gi0/0 (to HQ-CORE-R1)
    
- Gi0/2 (to HQ-ACC-SW-01)
    
- Gi0/3 (to HQ-DIST-SW-02)
    


```plaintext
enable
configure terminal
interface range Gi0/0,Gi0/2,Gi0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,99
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260110035434.png)

<br>

### **HQ-DIST-SW-02 - Trunk Ports**

Trunk ports on HQ-DIST-SW-02:

- Gi0/0 (to HQ-CORE-R2)
    
- Gi0/3 (to HQ-ACC-SW-02)
    
- Gi0/1 (to HQ-DIST-SW-01)
    

```plaintext
enable
configure terminal
interface range Gi0/0,Gi0/1,Gi0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,99
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260110035818.png)


<br>

### **HQ-ACC-SW-01 - Trunk Ports**

Trunk ports on HQ-ACC-SW-01:

- Gi0/1 (to HQ-DIST-SW-01)
    
- Gi0/3 (to HQ-ACC-SW-02)
    

```plaintext
enable
configure terminal
interface range Gi0/1,Gi0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,99
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260109153917.png)

<br>

### **HQ-ACC-SW-02 - Trunk Ports**

Trunk ports on HQ-ACC-SW-02:

- Gi0/2 (to HQ-DIST-SW-02)
    
- Gi0/3 (to HQ-ACC-SW-02)
    

```plaintext
enable
configure terminal
interface range Gi0/2,Gi0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,99
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260109153959.png)

<br>

## **4.9 Configure Trunk Ports - Branch-01 (BR1)**


This part configures trunk ports within the Branch-01 switching layer. Trunk links are used between switches and toward the branch edge router to carry multiple VLANs across the site and to support resilient and stable Layer 2 connectivity. Allowing all required branch VLANs on trunk links ensures consistent Layer 2 connectivity and enables traffic to continue flowing during link or device failures.

The switching devices in this project use an IOSv-L2 image, which requires explicit configuration of 802.1Q encapsulation on trunk ports. For this reason, trunk encapsulation is defined manually on all trunk interfaces.

<br>

### **BR1-DIST-SW-01 - Trunk Ports**

Trunk ports on BR1-DIST-SW-01:

- Gi0/0 (to BR1-DIST-SW-02)
    
- Gi0/1 (to BR1-EDGE-R4)
    

```plaintext
enable
configure terminal
interface range Gi0/0,Gi0/1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 50,60,70,80,99
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260109154040.png)

<br>

### **BR1-DIST-SW-02 - Trunk Ports**

Trunk ports on BR1-DIST-SW-02:

- Gi0/0 (to BR1-DIST-SW-01)
    
- Gi0/2 (to BR1-EDGE-R4)
    

```plaintext
enable
configure terminal
interface range Gi0/0,Gi0/2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 50,60,70,80,99
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260109154113.png)


<br>

### **Results - Trunk Configuration (HQ and BR1)**


All trunk ports are configured to carry the required VLANs across the switching layer at both sites. Trunk encapsulation is set to 802.1Q on all trunk interfaces to match the IOSv-L2 switching image requirements. The trunk configuration provides consistent Layer 2 connectivity and reliable VLAN forwarding across the network. The switching environment is now ready for spanning tree configuration.

<br>

## **4.10 Spanning Tree Configuration (RSTP) - HQ and BR1**

This next section configures Rapid Spanning Tree Protocol (RSTP) for the enterprise switching design. RSTP is used to prevent Layer 2 loops and to ensure stable and predictable Layer 2 topology behavior without creating broadcast storms. By controlling port states, RSTP maintains a loop-free switching environment across the network.

RSTP is enabled on all switches to ensure consistent spanning tree operation at each site. Root bridge roles are defined on distribution switches to control the Layer 2 topology in a deterministic manner. Access switches use the default spanning tree priority and follow the root selection defined at the distribution layer.

<br>


### **HQ-DIST-SW-01 - Enable RSTP and Set Root Primary**

```plaintext
enable
configure terminal
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,99 priority 4096
end
write memory
```
![](images/Pasted%20image%2020260109154208.png)

<br>

### **HQ-DIST-SW-02 - Enable RSTP and Set Root Secondary**

```plaintext
enable
configure terminal
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,99 priority 8192
end
write memory
```
![](images/Pasted%20image%2020260109154231.png)

<br>

### **HQ-ACC-SW-01 - Enable RSTP**

```plaintext
enable
configure terminal
spanning-tree mode rapid-pvst
end
write memory
```
![](images/Pasted%20image%2020260109154302.png)

<br>


### **HQ-ACC-SW-02 - Enable RSTP**

```plaintext
enable
configure terminal
spanning-tree mode rapid-pvst
end
write memory
```
![](images/Pasted%20image%2020260109154324.png)



<br>

## **4.11 Spanning Tree Configuration (RSTP) - Branch-01 (BR1)**

<br>

### **BR1-DIST-SW-01 - Enable RSTP and Set Root Primary**

```plaintext
enable
configure terminal
spanning-tree mode rapid-pvst
spanning-tree vlan 50,60,70,80,99 priority 4096
end
write memory
```
![](images/Pasted%20image%2020260109154346.png)

<br>


### **BR1-DIST-SW-02 - Enable RSTP and Set Root Secondary**

```plaintext
enable
configure terminal
spanning-tree mode rapid-pvst
spanning-tree vlan 50,60,70,80,99 priority 8192
end
write memory
```
![](images/Pasted%20image%2020260109154408.png)

<br>

### **Results - Spanning Tree Configuration (HQ and BR1)**

RSTP is enabled on all switches at both sites, and root bridge roles are set at the distribution layer using explicit bridge priorities. This prevents Layer 2 loops and keeps the switching topology stable and predictable. The spanning tree configuration creates a simple and resilient Layer 2 structure that fits the enterprise network design.

<br>

## **4.12 Final Verification**


This last section verifies the Layer 2 configuration after VLAN, access port, trunk, and RSTP setup. The verification confirms that VLANs are correctly created, trunk links are active, and interfaces are operating in the expected modes.

The command outputs also allow validation that PortFast and BPDU Guard are applied on access ports and that Rapid Spanning Tree Protocol (RSTP) is running correctly with the intended root bridge roles.

**For documentation purposes, verification is demonstrated on two example switches:**

- HQ-DIST-SW-01 (Headquarters distribution switch)
    
- BR1-DIST-SW-02 (Branch-01 distribution switch)
    

---

<br>

### **HQ-DIST-SW-01 - Verification**

    
#### Verification Commands

```plaintext
show vlan brief
show spanning-tree summary
```
![](images/Pasted%20image%2020260110043845.png)

<br>

#### Additional Verification Commands

```plaintext
show interfaces trunk
show interfaces status
```
![](images/Pasted%20image%2020260110044118.png)


---

<br>

### **BR1-DIST-SW-02 - Verification**

    
#### Verification Commands

```plaintext
show vlan brief
show spanning-tree summary
```
![](images/Pasted%20image%2020260109155238.png)

<br>

#### Additional Verification Commands

```plaintext
show interfaces trunk
show interfaces status
```
![](images/Pasted%20image%2020260109155528.png)

<br>

#### Verify PortFast and BPDU Guard

```plaintext
show spanning-tree interface gi0/1 detail
```
![](images/Pasted%20image%2020260109160437.png)

<br>

### **Results**

VLANs are present and active on the switching devices, trunk links are operational with the expected allowed VLAN lists, and RSTP is running with the defined root bridge roles. PortFast and BPDU Guard are applied on access ports, ensuring a stable and loop-free Layer 2 topology ready for upper-layer configuration.


<br>

## **4.13 Conclusion**


This chapter builds the complete Layer 2 switching foundation for the network. VLANs are created to separate different types of traffic, access ports are assigned based on connected devices, trunk links are configured to carry VLAN traffic across the topology, and RSTP is enabled to protect the network from loops and ensure a resilient Layer 2 topology.

With VLANs, trunks, access ports, PortFast, BPDU Guard, and RSTP correctly configured and verified, the Layer 2 environment is stable and consistent across both headquarters and branch sites. The switching layer is now fully prepared, and the next chapter can focus on inter-VLAN routing and Layer 3 configuration.
<br>

---


<br>

**Next Chapter:** [Inter VLAN Routing and VRRP](05-inter-vlan-routing-and-vrrp.md)
