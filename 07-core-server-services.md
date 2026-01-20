

<br>

# **7 - Core Server Services**

<br>

## **7.1 Introduction**

This chapter represents the seventh stage of the enterprise network configuration process and introduces core server services required for dynamic network operation across headquarters and branch locations.

The main focus is centralized DHCP, which provides automatic IP address assignment for multiple VLANs at both sites. By deploying DHCP on the infrastructure server, network management is simplified and all VLANs become fully operational with valid IP configuration, allowing connectivity to be verified across the network.

An internal HTTP service demonstrates access to local enterprise web resources across VLANs, while name resolution is provided using dnsmasq as a lightweight and less complex alternative to traditional DNS solutions. Centralized logging with rsyslog is introduced to demonstrate the importance of collecting system and network logs in one place for visibility and troubleshooting.

<br>

## **7.2 Topology Diagram**

![](images/Pasted%20image%2020260113070635.png)



<br>


## **7.3 Objectives**

1. Configure centralized DHCP on the infrastructure server, including creation of address scopes for headquarters and branch VLANs.
    
2. Configure an internal HTTP service to provide access to a local enterprise web resource across VLANs.
    
3. Implement basic name resolution using dnsmasq as a lightweight DNS solution for the internal network.
    
4. Centralized logging is implemented using rsyslog to collect system and network device logs on a single infrastructure server, improving troubleshooting, auditing, and overall network visibility.

<br>


## **7.4 DHCP Services**


This first section configures centralized DHCP services for the enterprise network. The DHCP service runs on the infrastructure server (HQ-SRV-INFRA-01) and provides dynamic IPv4 addressing for client VLANs at both headquarters and branch locations. Infrastructure, management, and monitoring server devices continue to use static IPv4 addressing.

DHCP requests are broadcast traffic and cannot cross VLAN segments by default. To support centralized DHCP, relay functionality is used on routing devices to forward client requests from client VLANs to the DHCP server.

### **DHCP Scope Overview**

<br>

### **Headquarters**

| **VLAN** | **VLAN Name** | **Subnet**    | **Gateway (VRRP)** | **DHCP Range**              | **Notes**       |
| -------- | ------------- | ------------- | ------------------ | --------------------------- | --------------- |
| 30       | Office        | 10.50.30.0/24 | 10.50.30.1         | 10.50.30.100 - 10.50.30.199 | Dynamic clients |
| 40       | Finance       | 10.50.40.0/24 | 10.50.40.1         | 10.50.40.100 - 10.50.40.199 | Dynamic clients |

### **Branch-01**

| **VLAN** | **VLAN Name** | **Subnet**    | **Gateway** | **DHCP Range**              | **Notes**               |
| -------- | ------------- | ------------- | ----------- | --------------------------- | ----------------------- |
| 50       | Office-01     | 10.60.50.0/24 | 10.60.50.1  | 10.60.50.100 - 10.60.50.199 | Dynamic clients         |
| 60       | Office-02     | 10.60.60.0/24 | 10.60.60.1  | 10.60.60.100 - 10.60.60.199 | Dynamic clients         |
| 70       | Sales         | 10.60.70.0/24 | 10.60.70.1  | 10.60.70.100 - 10.60.70.199 | Dynamic clients         |
| 80       | Printer       | 10.60.80.0/27 | 10.60.80.1  | N/A                         | Static device (printer) |


<br>

### **DHCP Server Installation (HQ-SRV-INFRA-01)**

Commands:

```prikaz
sudo apt update
sudo apt install isc-dhcp-server -y
```
![](images/Pasted%20image%2020260113002237.png)

Bind the service to the correct interface:

```prikaz
sudo nano /etc/default/isc-dhcp-server
```


Set IPv4 interface:

```prikaz
INTERFACESv4="ens3"
```
![](images/Pasted%20image%2020260113002343.png)

<br>

### **Server Routing Requirement**

The DHCP server requires a default route to reach routed client VLANs located behind Layer 3 devices.

Default route configuration:

```prikaz
sudo ip route add default via 10.50.10.1
```
![](images/Pasted%20image%2020260113002641.png)

<br>

### **DHCP Scope Configuration**

Edit DHCP configuration file:

```prikaz
sudo nano /etc/dhcp/dhcpd.conf
```

### DHCP Scopes - dhcpd.conf

##### Edit DHCP configuration file

```prikaz
sudo nano /etc/dhcp/dhcpd.conf
```


#### configuration

```prikaz
authoritative;

10

option domain-name "corp.local";
option domain-name-servers 10.50.10.10;

# VLAN 10 - Infrastructure server subnet (no DHCP pool)
subnet 10.50.10.0 netmask 255.255.255.192 {
}

# VLAN 20 - Monitoring subnet (no DHCP pool)
subnet 10.50.20.0 netmask 255.255.255.192 {
}

# VLAN 99 - Management subnet (no DHCP pool)
subnet 10.50.99.0 netmask 255.255.255.192 {
}

# VLAN 30 - Headquarters Office
subnet 10.50.30.0 netmask 255.255.255.0 {
  option routers 10.50.30.1;
  range 10.50.30.100 10.50.30.199;
}

# VLAN 40 - Headquarters Finance
subnet 10.50.40.0 netmask 255.255.255.0 {
  option routers 10.50.40.1;
  range 10.50.40.100 10.50.40.199;
}

# VLAN 99 - Branch-01 - Management (no DHCP pool)
subnet 10.60.99.0 netmask 255.255.255.192 {
}

# VLAN 50 - Branch-01 - Office-01
subnet 10.60.50.0 netmask 255.255.255.0 {
  option routers 10.60.50.1;
  range 10.60.50.100 10.60.50.199;
}

# VLAN 60 - Branch-01 - Office-02
subnet 10.60.60.0 netmask 255.255.255.0 {
  option routers 10.60.60.1;
  range 10.60.60.100 10.60.60.199;
}

# VLAN 70 - Branch-01 - Sales
subnet 10.60.70.0 netmask 255.255.255.0 {
  option routers 10.60.70.1;
  range 10.60.70.100 10.60.70.199;
}

# VLAN 80 - Branch-01 - Printer (no DHCP pool)
subnet 10.60.80.0 netmask 255.255.255.224 {
}
```
![](images/Pasted%20image%2020260113010254.png)
![](images/Pasted%20image%2020260113010312.png)


#### Restart the DHCP service.

```plaintext
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
```
![](images/Pasted%20image%2020260113010538.png)

<br>

### **DHCP Relay Configuration**

This subsection configures DHCP relay so that client VLANs can reach the central DHCP server (HQ-SRV-INFRA-01) across routed links. DHCP requests are broadcast by default, so Layer 3 gateway interfaces forward them to the DHCP server using `ip helper-address`.

**DHCP Server IPv4:**

- HQ-SRV-INFRA-01: 10.50.10.10
    

**Notes:**

- DHCP relay is configured only on VLAN gateway interfaces (SVI/subinterfaces) for VLANs that use DHCP.
    
- VLANs with static addressing (Infrastructure, Monitoring, Management, Printer) do not need relay.
    
- No relay is configured on HQ-EDGE-R3 because it does not provide VLAN gateway subinterfaces.
    

<br>

#### **HQ-CORE-R1 - DHCP Relay**

Relay is required only for HQ client VLANs:

- VLAN 30 (Headquarters Office)
    
- VLAN 40 (Headquarters Finance) 
    

```prikaz
enable
configure terminal
interface GigabitEthernet0/0.30
ip helper-address 10.50.10.10
exit
interface GigabitEthernet0/0.40
ip helper-address 10.50.10.10
exit
end
write memory
```
![](images/Pasted%20image%2020260113013730.png)

<br>

#### **HQ-CORE-R2 - DHCP Relay**

Same relay configuration as HQ-CORE-R1 for the HQ client VLANs.

```prikaz
enable
configure terminal
interface GigabitEthernet0/0.30
ip helper-address 10.50.10.10
exit
interface GigabitEthernet0/0.40
ip helper-address 10.50.10.10
exit
end
write memory
```
![](images/Pasted%20image%2020260113013748.png)

<br>

#### **BR1-EDGE-R4 - DHCP Relay**

Branch client VLANs using DHCP:

- VLAN 50 (Office-01)
    
- VLAN 60 (Office-02)
    
- VLAN 70 (Sales-01) 
    

```prikaz
enable
configure terminal
interface GigabitEthernet0/1.50
ip helper-address 10.50.10.10
exit
interface GigabitEthernet0/1.70
ip helper-address 10.50.10.10
exit
interface GigabitEthernet0/2.60
ip helper-address 10.50.10.10
exit
end
write memory
```
![](images/Pasted%20image%2020260113013839.png)

<br>

### **Results**

DHCP relay **works** on all L3 gateway interfaces for client VLANs in the HQ and the Branch. DHCP requests from VLAN 30 and 40 in the headquarters and from VLAN 50, 60, and 70 in the branch **go** to the central DHCP server (HQ-SRV-INFRA-01). All clients **get** their IPv4 addresses correctly.

VLANs with static addresses (Infrastructure, Monitoring, Management, and Printer) **do not use** DHCP relay. This **keeps** the network architecture clean and consistent.


<br>

### **Verification DHCP

<br>

##### **Example - Branch-01 VPC Client (VLAN 70 - Sales-01)**

Set the Branch-01 Sales client to use DHCP and confirm assigned settings.

```prikaz
ip dhcp
show ip
```
![](images/Pasted%20image%2020260113013921.png)

##### **Example - HQ VPC Client (VLAN 30 - Office-01)**

Set the Headquarters Office client to use DHCP and confirm assigned settings.

```prikaz
ip dhcp
show ip
```
![](images/Pasted%20image%2020260113013956.png)

<br>

#### **Example - Router DHCP Relay Quick Check (BR1-EDGE-R4)**

Confirm that DHCP relay

```prikaz
show running-config | include helper-address
```
![](images/Pasted%20image%2020260113014342.png)


### **Results:**  

DHCP works correctly for both Headquarters and Branch clients. VPC clients in VLAN 30 (HQ Office) and VLAN 70 (Branch Sales) successfully obtain IPv4 addresses, default gateways, and DNS information from the central DHCP server. DHCP relay is present only on the required router subinterfaces, confirming correct and clean relay configuration.


<br>

### **DHCP Relay Verification - Wireshark (Branch-01)**


This subsection verifies DHCP relay operation by monitoring DHCP traffic on the link between the Branch-01 edge router and the ISP. Packet capture is performed on interface Gi0/5 ↔ Gi0/5 to confirm that DHCP requests from Branch VLANs are correctly relayed as unicast traffic to the central DHCP server.

### **Capture Point**

- **Link:** Branch-01 EDGE ↔ ISP
    
- **Interface:** Gi0/5 – Gi0/5
    
![](images/Pasted%20image%2020260113015044.png)

### Capture Filter

```
bootp
```
![](images/Pasted%20image%2020260113015055.png)
### **Results:**

- DHCP Discover, Offer, Request, and ACK messages are visible on the monitored link.
    
- Traffic is observed as **unicast**, confirming correct DHCP relay behavior.
    
- Multiple DHCP transactions correspond to different Branch VLANs.
    
- Source addresses match Branch VLAN gateway interfaces, and the destination is the DHCP server.



<br>

## **7.5 HTTP Service (Apache2)**


This second section configures an Apache2 web server on the Xubuntu Infrastructure Server located in VLAN 10. The goal is to provide a simple internal web page that can be reached from all routed VLANs, including the monitoring network. HTTP is used as a clear Layer 7 verification of end-to-end connectivity.
sudo


### **Apache2 Installation**

Install the Apache2 web server package on the Infrastructure Server.

```prikaz
sudo apt update
sudo apt install apache2 -y
```
![](images/Pasted%20image%2020260113021253.png)


### Service Status and Local Verification

Verify that the Apache2 service is enabled and running, and that it listens on TCP port 80.

```prikaz
sudo systemctl status apache2 --no-pager -l
sudo systemctl is-enabled apache2
```
![](images/Pasted%20image%2020260113021333.png)

Local browser test (from the Infrastructure Server):

```prikaz
http://10.50.10.10
```
![](images/Pasted%20image%2020260113021431.png)


### **Replace the Default Apache Page**

Replace the default Apache page with a simple custom HTML page to clearly identify the project and server role.

```prikaz
sudo nano /var/www/html/index.html
```


Example content:

```plaintext
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>GNS3 Project - Enterprise Network</title>
  <style>
    body {
      background-color: #e6ffe6;
      font-family: Arial, sans-serif;
      margin: 40px;
    }
    h1 { color: #004d00; }
    p { font-size: 18px; }
  </style>
</head>
<body>
  <h1>GNS3 Project - Enterprise Network</h1>
  <p>This page is served from the Infrastructure Server in VLAN 10 (10.50.10.10).</p>
</body>
</html>
```
![](images/Pasted%20image%2020260113022712.png)

**Apply changes:**

```prikaz
ls -l /var/www/html/index.html
sudo systemctl reload apache2
```
![](images/Pasted%20image%2020260113022812.png)


### Client Verification - HQ-SRV-INFRA-01

Verify HTTP access from the Infrastructure environment.

```prikaz
http://10.50.10.10
```
![](images/Pasted%20image%2020260113022851.png)

### **Results**

The Apache2 web server responds correctly to HTTP requests. The custom intranet page loads successfully from both the Infrastructure and Monitoring networks, confirming correct routing, gateway configuration, and Layer 7 service availability across VLANs.

<br>

### **Wireshark Verify - HTTP Client Test from HQ-SRV-MON-01**


This verification captures HTTP traffic in Wireshark while HQ-SRV-MON-01 opens the intranet web page hosted on HQ-SRV-INFRA-01. The capture confirms that routed connectivity is working and that the HTTP application exchange is visible on the server-to-switch link.


- **Client**: HQ-SRV-MON-01 (Monitoring server)
    
- **Web server:** HQ-SRV-INFRA-01 (Apache2)
    
- **Capture point:** link between HQ-SRV-MON-01 and HQ-ACC-SW-01 (server access link)
    


![](images/Pasted%20image%2020260113023729.png)


#### **Show TCP port 80 (works even when HTTP is not decoded)**

```prikaz
tcp.port == 80
```
![](images/Pasted%20image%2020260113023428.png)

### **Results**

The Wireshark capture confirms a successful HTTP communication between the Monitoring Server (10.50.20.10) and the Infrastructure Web Server (10.50.10.10). The TCP three-way handshake (SYN, SYN-ACK, ACK) is completed, followed by an HTTP GET request and a valid **HTTP/1.1 200 OK** response carrying HTML content. The session is then properly closed with FIN/ACK packets. This verifies correct inter-VLAN routing, TCP connectivity on port 80, and a functioning Apache HTTP service reachable from the monitoring VLAN.

<br>

## **7.6 - DNS Service (dnsmasq)**


This next section configures a DNS service using dnsmasq on the Infrastructure Server (HQ-SRV-INFRA-01) in VLAN 10. The goal is to resolve internal hostnames in the corp.local domain so that clients and servers can reach services using names instead of IP addresses.

#### **DNS Design**

- **DNS Server:** HQ-SRV-INFRA-01 (10.50.10.10)
    
- **Domain:** corp.local
    
- **Primary Record (HTTP):** infra.corp.local -> 10.50.10.10
    

### **Install dnsmasq (HQ-SRV-INFRA-01)**

Commands:

```prikaz
sudo apt update
sudo apt install dnsmasq -y
```
![](images/Pasted%20image%2020260113025351.png)

<br>

### **Configure dnsmasq (HQ-SRV-INFRA-01)**

Edit the dnsmasq configuration file.

Commands:

```prikaz
sudo nano /etc/dnsmasq.conf
```


Configuration:

```prikaz
# Basic DNS settings
domain=corp.local
local=/corp.local/
expand-hosts

# Listen only on the server interface in VLAN 10
interface=ens3
bind-interfaces

# Local DNS records
address=/infra.corp.local/10.50.10.10

#Forward all non-local queries to upstream DNS
server=8.8.8.8
server=1.1.1.1
```
![](images/Pasted%20image%2020260113051639.png)


- domain and local define the internal DNS domain used by the network.
    
- interface and bind-interfaces limit dnsmasq to the VLAN 10 interface.
    
- address creates a simple local A record for the Infrastructure Server HTTP service.
    

### **Restart and Enable the Service**

Commands:

```prikaz
sudo systemctl enable dnsmasq
sudo systemctl restart dnsmasq
sudo systemctl status dnsmasq --no-pager -l
```
![](images/Pasted%20image%2020260113030044.png)

<br>

### **Verification - HQ-SRV-INFRA-01**

Verify DNS in browser from the HQ-SRV-INFRA-01

```prikaz
infra.corp.local
```
![](images/Pasted%20image%2020260113034124.png)

<br>

### **Static DNS Configuration (Monitoring Server)**


The monitoring server does not use DHCP by design. Because of this, DNS settings are not received automatically and must be configured manually. The default `/etc/resolv.conf` file is managed by `systemd-resolved` and must be replaced with a static configuration.

### **Configuration**

**Remove the systemd-resolved symlink**

Before creating a static DNS configuration, the existing `systemd-resolved` service and symlink must be removed.

```
sudo systemctl disable systemd-resolved  
sudo systemctl stop systemd-resolved  
sudo rm /etc/resolv.conf
```


##### **Create a static resolv.conf file**

```
sudo nano /etc/resolv.conf
```


**After:**

```
nameserver 10.50.10.10  
search corp.local
```
![](images/Pasted%20image%2020260113035651.png)





<br>

##### **Verify from HQ-SRV-MON-01**

Confirm that the Monitoring Server can resolve the name and reach the HTTP service.

Commands:

```prikaz
nslookup infra.corp.local 10.50.10.10
ping infra.corp.local
```
![](images/Pasted%20image%2020260113035827.png)


##### **Verify from BR-SALES-01**

Test connectivity to **infra.corp.local** from the Sales VLAN 60

![](images/Pasted%20image%2020260113040632.png)


### **Results**

dnsmasq runs on HQ-SRV-INFRA-01 and provides internal name resolution for the corp.local domain. HQ-SRV-MON-01 resolves infra.corp.local to 10.50.10.10 and reaches the Apache HTTP service using the hostname.


<br>



## **7.7 Rsyslog - Centralized Logging**


Rsyslog is used in this project to centralize log messages from network devices on a single infrastructure server. This allows administrators to detect network issues, interface failures, and redundancy events without direct access to individual devices. Centralized logging improves troubleshooting, auditing, and overall network visibility.



### **Rsyslog Installation and Configuration (HQ-SRV-INFRA-01)**

**Install the rsyslog service on the infrastructure server:**

```prikaz
sudo apt update
sudo apt install rsyslog -y
```
![](images/Pasted%20image%2020260113050039.png)

**Enable rsyslog to receive remote logs by editing the configuration file:**

```prikaz
sudo nano /etc/rsyslog.conf

```


**Ensure UDP reception is enabled:**

```prikaz
module(load="imudp")
input(type="imudp" port="514")
```
![](images/Pasted%20image%2020260113050432.png)

**Restart and enable the service:**

```prikaz
sudo systemctl status rsyslog --no-pager -l
```
![](images/Pasted%20image%2020260113055110.png)


<br>

### **Network Device Configuration (Example - HQ-CORE-R1)**

Configure the core router to send syslog messages to the infrastructure server:

```prikaz
enable
configure terminal
service timestamps log datetime msec
logging host 10.50.10.10
logging trap informational
logging on
exit
write memory
```
![](images/Pasted%20image%2020260113055650.png)

The same configuration principle applies to all other routers and switches in the network.


<br>

### **Verification (Infrastructure Server)**

Verify that log messages are received from network devices:

```prikaz
sudo tail -f /var/log/syslog
```
![](images/Pasted%20image%2020260113060404.png)



<br>


### **Scenario A - Interface Failure Detection**

A physical or administrative shutdown is performed **on interface Gi0/0 of HQ-CORE-R1**. The rsyslog server records an interface state change message, confirming immediate detection of link failures.

![](images/Pasted%20image%2020260113060632.png)

After shutting down the Gi0/0 interface on the primary core router, VRRP state changes were logged on the infrastructure server.  
The backup router successfully transitioned to the MASTER role for all VRRP groups, ensuring uninterrupted gateway availability.


<br>

### **Scenario B – Centralized DHCP Lease Logging**

**Clients in VLANs 40, 50, 60 request IP configuration via DHCP**.  
The DHCP service is provided by the central Infrastructure Server (**HQ-SRV-INFRA-01**). During the address assignment process, DHCP events (DISCOVER, OFFER, REQUEST, ACK) are logged on the infrastructure server via rsyslog, demonstrating centralized visibility into client network onboarding across multiple VLANs.


![](images/Pasted%20image%2020260113061359.png)


DHCP requests from clients in VLANs 40, 50, and 60 were successfully processed by the central DHCP server (HQ-SRV-INFRA-01).  
The infrastructure server logged the full DHCP transaction lifecycle (DISCOVER, OFFER, REQUEST, ACK) for multiple clients across different VLANs. This confirms correct DHCP relay configuration, centralized address management, and functional rsyslog-based event logging across the enterprise network.

<br>

### **Results**

Centralized logging successfully captures interface failures and redundancy events from network devices. Rsyslog provides real-time visibility into network behavior and supports effective troubleshooting and monitoring in the enterprise network.

<br>


## **7.8 Conclusion**

In this chapter, core infrastructure services were deployed and tested within the enterprise network. DHCP successfully assigns IP addresses across multiple VLANs, internal DNS resolution functions correctly, and centralized syslog logging captures important network events and state changes. Overall network connectivity is stable and all essential services operate as expected. In the next chapter, NAT and PAT will be configured to provide internet access by translating internal private addresses to public addresses for both sites.





<br>

---


<br>

**Next Chapter:**

