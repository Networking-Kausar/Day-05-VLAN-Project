# CCNA Lab Day 5 – Enterprise VLAN Project

## Overview

This lab combines all switching and routing concepts learned in the previous labs into a small enterprise network. The project includes VLAN creation, trunking, Native VLAN configuration, Allowed VLANs, and Router-on-a-Stick for Inter-VLAN Routing.

The goal is to build a functional enterprise network where multiple departments can communicate securely while remaining logically separated.

## Objectives

After completing this lab, you will be able to:

- Create and manage VLANs
- Configure access ports
- Configure trunk links
- Configure Native VLANs
- Configure Allowed VLANs
- Implement Router-on-a-Stick
- Enable Inter-VLAN Routing
- Verify end-to-end connectivity
- Troubleshoot enterprise network issues

---

## Network Topology

```text
                     R1
                  G0/0
                    |
                    |
              Fa0/1 (TRUNK)
                    |
                  SW1
                    |
         Fa0/24==========Fa0/24
              TRUNK LINK
                    |
                  SW2


VLAN 10 (HR)              VLAN 30 (SALES)
PC1                       PC3
Fa0                       Fa0
 |                          |
SW1 Fa0/2               SW2 Fa0/2


VLAN 20 (FINANCE)         VLAN 40 (IT)
PC2                       PC4
Fa0                       Fa0
 |                          |
SW1 Fa0/3               SW2 Fa0/3
```

---

## Devices Used

- 1 Cisco Router (R1)
- 2 Cisco Switches (SW1, SW2)
- 4 PCs
- Copper Straight-Through Cables

---

## Physical Connections

### Router Connections

| Device | Interface | Connected To |
|----------|----------|----------|
| R1 | G0/0 | SW1 Fa0/1 |

### Switch-to-Switch Trunk

| Device | Interface | Connected To |
|----------|----------|----------|
| SW1 | Fa0/24 | SW2 Fa0/24 |

### End Devices

| Device | Interface | Connected To |
|----------|----------|----------|
| PC1 | Fa0 | SW1 Fa0/2 |
| PC2 | Fa0 | SW1 Fa0/3 |
| PC3 | Fa0 | SW2 Fa0/2 |
| PC4 | Fa0 | SW2 Fa0/3 |

---

## VLAN Assignment

| VLAN ID | VLAN Name | Device |
|----------|----------|----------|
| 10 | HR | PC1 |
| 20 | FINANCE | PC2 |
| 30 | SALES | PC3 |
| 40 | IT | PC4 |
| 999 | NATIVE | Trunk Links |

---

## IP Addressing

| Device | IP Address | Subnet Mask | Default Gateway |
|----------|----------|----------|----------|
| PC1 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC2 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |
| PC3 | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 |
| PC4 | 192.168.40.10 | 255.255.255.0 | 192.168.40.1 |

---

# SW1 Configuration

## Create VLANs

```bash
enable
configure terminal

vlan 10
name HR

vlan 20
name FINANCE

vlan 30
name SALES

vlan 40
name IT

vlan 999
name NATIVE
```

## Configure Access Ports

```bash
interface fa0/2
switchport mode access
switchport access vlan 10

interface fa0/3
switchport mode access
switchport access vlan 20
```

## Configure Trunk Toward Router

```bash
interface fa0/1
switchport mode trunk

switchport trunk native vlan 999

switchport trunk allowed vlan 10,20,30,40,999
```

## Configure Trunk Toward SW2

```bash
interface fa0/24
switchport mode trunk

switchport trunk native vlan 999

switchport trunk allowed vlan 10,20,30,40,999
```

---

# SW2 Configuration

## Create VLANs

```bash
enable
configure terminal

vlan 10
name HR

vlan 20
name FINANCE

vlan 30
name SALES

vlan 40
name IT

vlan 999
name NATIVE
```

## Configure Access Ports

```bash
interface fa0/2
switchport mode access
switchport access vlan 30

interface fa0/3
switchport mode access
switchport access vlan 40
```

## Configure Trunk Toward SW1

```bash
interface fa0/24

switchport mode trunk

switchport trunk native vlan 999

switchport trunk allowed vlan 10,20,30,40,999
```

---

# Router Configuration

## Enable Physical Interface

```bash
enable
configure terminal

interface g0/0
no shutdown
```

### VLAN 10 Gateway

```bash
interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
```

### VLAN 20 Gateway

```bash
interface g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
```

### VLAN 30 Gateway

```bash
interface g0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
```

### VLAN 40 Gateway

```bash
interface g0/0.40
encapsulation dot1Q 40
ip address 192.168.40.1 255.255.255.0
```

---

## Verification Commands

### Verify VLAN Configuration

```bash
show vlan brief
```

### Verify Trunk Links

```bash
show interfaces trunk
```

### Verify Router Interfaces

```bash
show ip interface brief
```

### Verify Routing Table

```bash
show ip route
```

---

## Connectivity Testing

### VLAN Gateway Tests

```bash
PC1 -> 192.168.10.1
PC2 -> 192.168.20.1
PC3 -> 192.168.30.1
PC4 -> 192.168.40.1
```

Expected Result:

Success

### Inter-VLAN Tests

```bash
PC1 -> PC2
PC1 -> PC3
PC1 -> PC4
PC2 -> PC4
PC3 -> PC4
```

Expected Result:

Success

---

## Traffic Flow Example

### PC3 to PC1

```text
PC3
 |
SW2
 |
Trunk Link
 |
SW1
 |
Router R1

Routing Decision

R1
 |
SW1
 |
PC1
```

The router receives VLAN 30 traffic and routes it to VLAN 10.

---

## Troubleshooting Scenarios

### Scenario 1: VLAN 30 Removed from Trunk

```bash
interface fa0/24
switchport trunk allowed vlan 10,20,40,999
```

Result:

- PC3 loses connectivity
- VLAN 30 traffic cannot cross the trunk

### Scenario 2: Native VLAN Mismatch

SW1:

```bash
switchport trunk native vlan 999
```

SW2:

```bash
switchport trunk native vlan 100
```

Result:

- Native VLAN mismatch warning
- Potential traffic issues

### Scenario 3: Incorrect Default Gateway

Configure PC4:

```text
Default Gateway = 192.168.40.254
```

Result:

- PC4 cannot communicate with other VLANs

### Scenario 4: Router Interface Shutdown

```bash
interface g0/0
shutdown
```

Result:

- Inter-VLAN Routing stops completely

---

## Mini Challenge

Create a new department:

### VLAN 50 – MANAGEMENT

Requirements:

- Create VLAN 50 on both switches
- Configure a new PC
- Create Router Subinterface G0/0.50
- Assign IP Address 192.168.50.1/24
- Allow VLAN 50 on all trunk links
- Verify communication with all VLANs

---

## Skills Learned

- VLAN Creation and Management
- Access Port Configuration
- Trunk Configuration
- Native VLAN Configuration
- Allowed VLAN Configuration
- Router-on-a-Stick
- Inter-VLAN Routing
- Enterprise Network Design
- Network Verification
- Troubleshooting Techniques

---

## Conclusion

This lab combines multiple CCNA switching and routing concepts into a single enterprise network project. By completing this lab, you gain hands-on experience with VLANs, trunking, Native VLANs, Router-on-a-Stick, and Inter-VLAN Routing while practicing troubleshooting skills commonly used in real-world environments.

---

## Next Lab

### Day 6 – Spanning Tree Protocol (STP)

Topics:

- Layer 2 Loops
- Broadcast Storms
- Root Bridge Election
- Port Roles
- Port States
- STP Verification
- STP Troubleshooting

---

## Author

Muhammad Kausar

CCNA Enterprise Networking Lab Series
