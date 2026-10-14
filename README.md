# Multi-Router Network: OSPF, DHCP Relay and PAT
<img width="672" height="326" alt="Screenshot 2026-10-15 002421" src="https://github.com/user-attachments/assets/ce3a5639-cfb7-4409-a0c2-14f8fcd3c947" />


## Overview

A PingMyNetwork Sandbox lab with five routers (R1, R3, R4, R5 and ISP), three switches (SW1–SW3), three PCs and a central DHCP server.

R1, R3 and R5 each connect to a user LAN and exchange routes with each other using OSPF. The DHCP server sits on its own network behind R4, so the routers facing the PC LANs relay DHCP requests to it. R4 connects the network to the ISP router.

**Purpose:** practice building a small multi-router network where internal routing, centralized DHCP and an internet-facing edge router work together.

## Technologies Configured

- **OSPF** (process 1, area 0) on R1, R3 and R5
- **DHCP relay** (`ip helper-address`) on R1, R3 and R5

## Addressing

| Link / LAN | Network | Devices |
|---|---|---|
| R1 – R3 | 192.168.10.0/24 | R1 `.1`, R3 `.2` |
| R1 – R5 | 192.168.11.0/24 | R1 `.1`, R5 `.2` |
| PC2 LAN | 192.168.12.0/24 | R1 `.1` |
| R3 – R4 | 192.168.13.0/24 | R3 `.1`, R4 `.2` |
| PC1 LAN | 192.168.14.0/24 | R3 `.1` |
| R4 – R5 | 192.168.15.0/24 | R5 `.2` |
| DHCP server network | 192.168.16.0/24 | DHCP server `.3` (R4 side) |
| PC3 LAN | 192.168.17.0/24 | R5 `.1` |
| R4 – ISP | 15.140.31.0/24 | R4 `.91`, ISP `.92` |

## Router Configurations

Full `show run` output for each device is in the [`New folder`](New%20folder) directory.

### R1 – [R1.txt](New%20folder/R1.txt)

| Interface | IP Address | Notes |
|---|---|---|
| Gi0/0 | 192.168.10.1/24 | Link to R3 |
| Gi0/1 | 192.168.12.1/24 | PC2 LAN, DHCP relay |
| Gi0/2 | 192.168.11.1/24 | Link to R5 |

```
interface GigabitEthernet0/1
 ip helper-address 192.168.16.3
!
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.11.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.255 area 0
```

### R3 – [R3.txt](New%20folder/R3.txt)

| Interface | IP Address | Notes |
|---|---|---|
| Gi0/0 | 192.168.14.1/24 | PC1 LAN, DHCP relay |
| Gi0/1 | 192.168.13.1/24 | Link to R4 |
| Gi0/2 | 192.168.10.2/24 | Link to R1 |

```
interface GigabitEthernet0/0
 ip helper-address 192.168.16.3
!
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.13.0 0.0.0.255 area 0
 network 192.168.14.0 0.0.0.255 area 0
```

### R5 – [R5.txt](New%20folder/R5.txt)

| Interface | IP Address | Notes |
|---|---|---|
| Gi0/0 | 192.168.11.2/24 | Link to R1 |
| Gi0/1 | 192.168.15.2/24 | Link to R4 |
| Gi0/2 | 192.168.17.1/24 | PC3 LAN, DHCP relay |

```
interface GigabitEthernet0/2
 ip helper-address 192.168.16.3
!
router ospf 1
 network 192.168.11.0 0.0.0.255 area 0
 network 192.168.15.0 0.0.0.255 area 0
 network 192.168.17.0 0.0.0.255 area 0
```

### R4 (edge router) – [R4.txt](New%20folder/R4.txt)

| Interface | IP Address | Notes |
|---|---|---|
| Gi0/0 | 15.140.31.91/24 | Link to ISP |
| Gi0/1 | 192.168.13.2/24 | Link to R3 |

The captured R4 output ends partway through the interface list (at `GigabitEthernet0/2`).

### ISP – [ISP.txt](New%20folder/ISP.txt)

| Interface | IP Address | Notes |
|---|---|---|
| Gi0/0 | 15.140.31.92/24 | Link to R4 |

All other ISP interfaces are shut down.

## Verification

The evidence included in this repository is the `show run` output captured on each router, which confirms the interface addressing, OSPF `network` statements and DHCP relay addresses listed above:

```
R1# show run
R3# show run
R4# show run
R5# show run
ISP# show run
```

## Files

| File | Description |
|---|---|
| `New folder/Screenshot 2026-10-15 002421.png` | Network topology |
| `New folder/R1.txt`, `R3.txt`, `R4.txt`, `R5.txt`, `ISP.txt` | `show run` output from each router |
