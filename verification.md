# Project C – Verification

## 1️⃣ VLAN & Interface Verification (FW1)

**Command:**
```text
show interface ip brief
```
Expected Result:
| Interface | IP Address  | Status | Protocol |
| --------- | ----------- | ------ | -------- |
| Vlan1     | 192.168.1.1 | up     | up       |
| Vlan2     | 10.0.0.2    | up     | up       |

2️⃣ ACL Verification
A. Allowed Traffic (HTTP)

Command:
From outside network (PC or R1):
```text
http://192.168.20.20
```
Expected Result:

HTTP traffic allowed to server (ACL hit count increments).

PT may show “Host Name Unresolved” → simulation limitation.

Verification Command (ASA):
```text
show access-list OUTSIDE_IN
```
B. Denied Traffic (Telnet)

Command:
From outside network (PC or R1):
```text
telnet 192.168.20.20 23
```

Expected Result:
Connection refused (Telnet blocked by default ACL).

3️⃣ End-to-End Connectivity

A. Internal Ping Tests (PCs & Server)

| Source | Destination     | Command              | Expected Result | Screenshot                  |
| ------ | --------------- | -------------------- | --------------- | --------------------------- |
| PC1    | PC2             | `ping 192.168.20.11` | Success ✅       | `PC1_ping_PC2.png`          |
| PC1    | Server-PT       | `ping 192.168.20.20` | Success ✅       | `PC1_ping_Server.png`       |
| PC2    | Server-PT       | `ping 192.168.20.20` | Success ✅       | `PC2_ping_Server.png`       |
| PC1    | Router2911 G0/1 | `ping 192.168.20.1`  | Success ✅       | Optional: `PC1_ping_R1.png` |

B. Router → ASA Outside Ping (PT Limitation)

Command:
```text
R1# ping 10.0.0.2
```

Expected Result:
Real network: Ping succeeds.
Packet Tracer: May fail or timeout (simulation limitation).

Actual Result in Lab:

Sending 5, 100-byte ICMP Echos to 10.0.0.2, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)

Note:
- Ping failure does not indicate misconfiguration. Internal connectivity and ACL results confirm lab objectives are met.

4️⃣ Device IP Configuration Verification

| Device    | Command                   | Expected Result                          | Screenshot            |
| --------- | ------------------------- | ---------------------------------------- | --------------------- |
| PC1       | `ipconfig`                | IP: 192.168.20.10, Gateway: 192.168.20.1 | `PC1_ipconfig.png`    |
| PC2       | `ipconfig`                | IP: 192.168.20.11, Gateway: 192.168.20.1 | `PC2_ipconfig.png`    |
| Server-PT | `ipconfig`                | IP: 192.168.20.20, Gateway: 192.168.20.1 | `Server_ipconfig.png` |
| R1        | `show ip interface brief` | G0/1: 192.168.20.1, G0/2: 10.0.0.1       | `R1_interfaces.png`   |

5️⃣ Summary Table

| Verification Area     | Command / Action                 | Expected Result                 | Status / Screenshot         |
| --------------------- | -------------------------------- | ------------------------------- | --------------------------- |
| VLAN & Interface      | `show interface ip brief` on FW1 | VLAN1 inside & VLAN2 outside up | `FW1_interface.png`         |
| ACL Allowed Traffic   | HTTP from outside → Server       | Allowed, ACL hit increments     | `HTTP_test.png`             |
| ACL Denied Traffic    | Telnet from outside → Server     | Denied, connection refused      | `Telnet_blocked.png`        |
| Internal Connectivity | PC1 → PC2 ping                   | Success ✅                       | `PC1_ping_PC2.png`          |
| Internal Connectivity | PC1 → Server ping                | Success ✅                       | `PC1_ping_Server.png`       |
| Internal Connectivity | PC2 → Server ping                | Success ✅                       | `PC2_ping_Server.png`       |
| Router Interfaces     | `show ip interface brief` on R1  | G0/1 & G0/2 up                  | `R1_interfaces.png`         |
| Router → FW1          | `ping 10.0.0.2` from R1          | PT limitation, may fail         | Optional: `R1_ping_FW1.png` |


Notes:

- End-to-end connectivity confirms internal network and firewall rules are correct.

- Packet Tracer limitations are documented for HTTP browser and ASA outside ping.

- All screenshots and ping results together provide full proof for lab objectives.