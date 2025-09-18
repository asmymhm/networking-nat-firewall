# Week 3 — Project C: NAT & Edge Firewall

This lab demonstrates **internal network connectivity and edge firewall configuration** using Cisco devices in Packet Tracer.  
**Author:** <MH. Mohamed Asmy>

**Date:** <2025-09-18>

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---
Table of Contents

1.  [Project Overview](#project-overview)

2.  [Lab Topology](#lab-topology)

3.  [IP Addressing Table](#ip-addressing-table)

4.  [Device Configuration](#device-configuration)
    - [Router Configuration](#router-configuration)
    - [PC Configuration](#pc-configuration)
    - [Server Configuration](#server-configuration)
    - [Firewall Configuration](#firewall-configuration)

5.  [Lab Steps](#lab-steps)

6.  [Verification](#verification)
      - [VLAN & Interface Verification](#vlan--interface-verification)
      - [ACL Verification (Allowed vs Denied Traffic)](#acl-verification-allowed-vs-denied-traffic)
      - [End-to-End Connectivity](#end-to-end-connectivity)

7.  [Folder Structure](#folder-structure)

8.  [License](#license)

---

## Project Overview

| Item         | Details                                                                                                                                                              |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Project Name | NAT and Edge Firewall Lab                                                                                                                                            |
| Objective    | Configure NAT on the router and implement edge firewall rules on ASA5505 to secure network traffic while allowing internal PCs to access external network resources. |
| Devices Used | Router X2 (2911, 2811), Switch X1 (2960), PCs X2, Server-PT X1, ASA5505 Firewall                                                                                     |
| Software     | Cisco Packet Tracer (Windows Environment)                                                                                                                            |
| Key Concepts | NAT (Static/Dynamic), PAT, Access Control Lists (ACLs), Edge Firewall Configuration, End-to-End Connectivity Verification                                            |

Limitation:
Packet Tracer ASA5505 cannot perform NAT/PAT via CLI or GUI. Therefore, traffic from internal PCs to the outside network (Router 2811 / Internet) cannot be demonstrated.


---

## Lab Topology

<img src="diagrams\topology.png" alt="NAT PAT TOPOLOGY" style="border:1px solid #ddd; padding:5px; max-width:100%; height:auto;">

Physical Cabling Table

| Device      | Interface | Connected To | Device Interface |
| ----------- | --------- | ------------ | ---------------- |
| Router 2911 | G0/0      | Switch 2960  | Fa0/1            |
| Router 2911 | G0/1      | Server-PT    | Fa0              |
| Router 2911 | G0/2      | ASA5505      | Et0/0            |
| Switch 2960 | Fa0/2     | PC1          | NIC              |
| Switch 2960 | Fa0/3     | PC2          | NIC              |
| Switch 2960 | Fa0/4     | ASA5505      | Et0/2            |
| ASA5505     | Et0/1     | Router 2811  | Fa0/0            |


---

## IP Addressing Table

| Device      | Interface | IP Address    | Subnet Mask     | Default Gateway |
| ----------- | --------- | ------------- | --------------- | --------------- |
| PC1         | NIC       | 192.168.10.10 | 255.255.255.0   | 192.168.10.1    |
| PC2         | NIC       | 192.168.10.11 | 255.255.255.0   | 192.168.10.1    |
| Server-PT   | Fa0       | 192.168.20.10 | 255.255.255.0   | 192.168.20.1    |
| Router 2911 | G0/0      | 192.168.10.1  | 255.255.255.0   | –               |
| Router 2911 | G0/1      | 192.168.20.1  | 255.255.255.0   | –               |
| Router 2911 | G0/2      | 10.0.0.1      | 255.255.255.252 | –               |
| ASA5505     | Et0/0     | 10.0.0.2      | 255.255.255.252 | –               |
| ASA5505     | Et0/1     | 172.16.0.2    | 255.255.255.0   | –               |
| Router 2811 | Fa0/0     | 172.16.0.1    | 255.255.255.0   | –               |


---

## Device Configuration

- Router: `router-configs/R1.cfg`  
- Firewall: `firewall-configs/FW1.cfg`  
- PC: `pc-configs/PC.cfg` 
- Server: `server-configs/Server.txt` 

---
### Router Configuration

```text
enable
configure terminal
```
! Hostname
```text
hostname R1
```

! Inside LAN interface
```text
interface GigabitEthernet0/1
 description Inside LAN
 ip address 192.168.20.1 255.255.255.0
 no shutdown
 ```

! Outside interface towards ASA
```text
interface GigabitEthernet0/2
 description Link to ASA (outside)
 ip address 10.0.0.1 255.255.255.252
 no shutdown
 ```

! Default route: send unknown traffic to ASA
````text
ip route 0.0.0.0 0.0.0.0 10.0.0.2
````

```text
end
write memory
```

---
### PC Configuration

| Device | IP Address    | Subnet Mask   | Default Gateway |
| ------ | ------------- | ------------- | --------------- |
| PC1    | 192.168.10.10 | 255.255.255.0 | 192.168.10.1    |
| PC2    | 192.168.10.11 | 255.255.255.0 | 192.168.10.1    |

---
### Server Configuration

| Device    | IP Address    | Subnet Mask   | Default Gateway |
| --------- | ------------- | ------------- | --------------- |
| Server-PT | 192.168.20.20 | 255.255.255.0 | 192.168.20.1    |

---
### Firewall Configuration

```text
enable
configure terminal
hostname FW1
```

! VLAN Interfaces
```text
interface Vlan1
 nameif inside
 security-level 100
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 ```
 ```text
interface Vlan2
 nameif outside
 security-level 0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
 ```

! Assign Ethernet Ports to VLANs
```text
interface Ethernet0/0
 switchport access vlan 2   ! outside link to R1
 no shutdown
 ```
 ```text
interface Ethernet0/1
 switchport access vlan 1   ! internal / management
 no shutdown
 ```
 ```text
interface Ethernet0/2
 switchport access vlan 1   ! internal / management
 no shutdown
 ```
! Access Control Rules
! Allow outside HTTP traffic to internal Server (192.168.20.10)
```text
access-list OUTSIDE_IN extended permit tcp any host 192.168.20.10 eq www
```
! Apply ACL to outside interface
```text
access-group OUTSIDE_IN in interface outside
```
! Notes

! NAT / PAT not configured here
! because Packet Tracer ASA does not support full NAT features.
! R1 provides inside routing, ASA enforces firewall ACL.
! End-to-end testing limited to:
!   - ICMP ping tests
!   - HTTP allowed
!   - Telnet denied
```text
write memory
```
---

## Lab Steps

1. Configure routers, ASA, PCs, and server as above.

2. Assign VLANs and IP addresses.

3. Configure ACL on ASA for allowed traffic (TCP port 80).

4. Verify VLANs, interfaces, and connectivity inside.

5. Test allowed traffic (HTTP) and denied traffic (Telnet / TCP23) from outside.

6. Save all configurations.
---
## Verification

### VLAN & Interface Verification

Purpose: Verify VLANs are up and IP addresses are correct.

Expected Results:

ACL allows TCP port 80 to 192.168.20.10

All other traffic blocked by default

Hit count increments after HTTP test
```text
show interface ip brief
```

<img src="screenshots\FW1_interface.png" alt="NAT PAT TOPOLOGY" style="border:1px solid #ddd; padding:5px; max-width:100%; height:auto;">

---
### ACL Verification (Allowed vs Denied Traffic)

Test Allowed Traffic (HTTP)

Command (from outside network / PC or R1):

Expected Result: HTTP page loads successfully
```text
http://192.168.20.10
```
<img src="screenshots\HTTP_test.png" alt="HTTP TEST" style="border:1px solid #ddd; padding:5px; max-width:100%; height:auto;">

Test Denied Traffic (Telnet)

Command (from outside network / PC or R1):

Expected Result: Connection refused ❌
```text
telnet 192.168.20.10 23
```
<img src="screenshots\Telnet_blocked.png" alt="HTTP TEST" style="border:1px solid #ddd; padding:5px; max-width:100%; height:auto;">

---
### End-to-End Connectivity

Test internal network connectivity to confirm PCs, Server, and Router communicate correctly.

Tests to perform:

| Source Device   | Destination Device | Command              | Expected Result | 
| --------------- | ------------------ | -------------------- | ----------------| 
| PC1             | PC2                | `ping 192.168.10.11` | Success ✅                             
| PC1             | Server-PT          | `ping 192.168.20.20` | Success ✅                    
| PC2             | Server-PT          | `ping 192.168.20.20` | Success ✅                   
| PC1             | Router2911 G0/1    | `ping 192.168.20.1`  | Success ✅                    
| Router2911 G0/2 | FW1 outside        | `ping 10.0.0.2`      | Timeout

PC1 to PC2 
```text
ping 192.168.10.11
```
<img src="screenshots\PC1_ping_PC2.png" alt="PING TEST" style="border:1px solid #ddd; padding:5px; max-width:100%; height:auto;">

PC1 to Server-PT
```text
ping 192.168.20.20
```
<img src="screenshots\PC1_ping_Server.png" alt="PING TEST" style="border:1px solid #ddd; padding:5px; max-width:100%; height:auto;">

PC2 to Server-PT
```text
ping 192.168.20.20
```
<img src="screenshots\PC2_ping_Server.png" alt="PING TEST" style="border:1px solid #ddd; padding:5px; max-width:100%; height:auto;">

PC1 to R1
```text
ping 192.168.20.1
```
<img src="screenshots\PC1_ping_R1.png" alt="PING TEST" style="border:1px solid #ddd; padding:5px; max-width:100%; height:auto;">

R1 to FW1 outside
```text
ping 10.0.0.2
```
<img src="screenshots\R1_ping_FW1.png" alt="PING TEST" style="border:1px solid #ddd; padding:5px; max-width:100%; height:auto;">

---
## Folder Strcture

networking-nat-firewall/
├── README.md
├── labs
├── diagrams
│ └── topology.png
├── router-configs/
│ └── R1.cfg
├── pc-configs/
│ ├── PC.cfg
├── server-configs/
│ └── Server.cfg
├── firewall-configs/
│ └── FW1.cfg
├── screenshots/
│ ├── FW1_interface.png
│ ├── HTTP_test.png
│ ├── PC1_ping_PC2.png
│ ├── PC1_ping_R1.png
│ ├── PC1_ping_Server.png
│ └── PC2_ping_Server.png
│ └── R1_ping_FW1.png
│ └── Telnet_blocked.png
├── verification.md
└── LICENSE

---

## License


<a href="LICENSE">
  <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="MIT License" />
</a>

This project is licensed under the [MIT License](LICENSE).

---
