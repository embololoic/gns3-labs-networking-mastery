# TP01 Advanced - VTP + STP Rapid-PVST+ Load-Balancing

## 🎯 Lab Objectives
Layer 2 multi-VLAN network demonstrating:
- **VTP** automatic VLAN propagation (Cisco proprietary).
- **802.1Q Trunks** Native VLAN 99 management.
- **Rapid-PVST+** loop prevention + per-VLAN load-balancing.
- **STP Security** (PortFast, BPDU Guard).
- **Inter-VLAN Routing** Router-on-a-Stick.

**VLANs**: 99 (mgmt), 100, 200, 300, 400 (data).

## 🗺️ Topology & Addressing

![alt text](<Capture d'écran 2026-02-18 173758.png>)

**Switch Roles**:
- **SW1**: VTP Server + Root VLAN 100/200
- **SW4**: Root VLAN 300/400
- **SW2/SW3**: VTP Client
- **SW5-SW10**: Access (single gi0/0 trunk)

**Addressing**:
| VLAN | Subnet         | **Gateway (R1)** | SVI Mgmt (SW#)  |
|------|----------------|------------------|-----------------|
| 99   | 192.168.99.0/24| 192.168.99.1    | 192.168.99.X   |
| 100  | 10.0.100.0/24  | **10.0.1.1**    | SW1=.1         |
| 200  | 10.0.200.0/24  | **10.0.2.1**    | SW2=.2         |
| 300  | 10.0.300.0/24  | **10.0.3.1**    | SW3=.3         |
| 400  | 10.0.400.0/24  | **10.0.4.1**    | SW4=.4         |

## 📋 Configuration Steps

### 1. VTP Configuration
**Principle**: Centralized VLAN management via VTP Server/Client.

**SW1 (Server)**:
```cisco
vtp domain TP01
vtp mode server
vtp version 2
vtp password vtp123
vtp pruning
vlan 99,100,200,300,400
```

**Clients (SW2-SW10)**:
```cisco
vtp domain TP01
vtp mode client
vtp version 2
vtp password vtp123
```

**Verify**:
```cisco
show vtp status
show vlan brief
```
*Note*: Cisco proprietary. Alternatives: MVRP (IEEE 802.1ak).

### 2. 802.1Q Trunks
**Distribution Layer** (full mesh):
```cisco
interface range gi0/0-3, gi1/0-3, gi3/0
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 99,100,200,300,400
```

**Access Layer** (SW5-SW10):
```cisco
! Single trunk uplink
interface gi0/0
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 99,100,200,300,400

! Server access ports
interface range gi0/1-3
 switchport mode access
 switchport access vlan 100  ! Adapt per VLAN
```

**Verify**:
```cisco
show interfaces trunk
show cdp neighbors detail  # No Native VLAN Mismatch
```

### 3. Rapid-PVST+ & Load-Balancing
**Principle**: Per-VLAN spanning tree prevents loops + distributes traffic.

**Global**:
```cisco
spanning-tree mode rapid-pvst
```

**Root Assignment**:
```cisco
! SW1: Primary VLAN 100/200 (10.0.1.x/10.0.2.x)
spanning-tree vlan 100,200 root primary
spanning-tree vlan 300,400 root secondary

! SW4: Primary VLAN 300/400 (10.0.3.x/10.0.4.x)
spanning-tree vlan 300,400 root primary
spanning-tree vlan 100,200 root secondary
```

**Edge Security** (Access Layer):
```cisco
interface range gi0/1-3
 spanning-tree portfast
 spanning-tree bpduguard enable
```

**Verify**:
```cisco
show spanning-tree vlan 100 brief  # Root=SW1
show spanning-tree vlan 300 brief  # Root=SW4
```

### 4. Inter-VLAN Routing (R1)
```cisco
interface gi0/0.99
 encapsulation dot1Q 99 native
 ip address 192.168.99.1 255.255.255.0

interface gi0/0.100
 encapsulation dot1Q 100
 ip address 10.0.1.1 255.255.255.0

interface gi0/0.200
 encapsulation dot1Q 200
 ip address 10.0.2.1 255.255.255.0

interface gi0/0.300
 encapsulation dot1Q 300
 ip address 10.0.3.1 255.255.255.0

interface gi0/0.400
 encapsulation dot1Q 400
 ip address 10.0.4.1 255.255.255.0
```

### 5. Management SVIs
```cisco
interface vlan 99
 ip address 192.168.99.X 255.255.255.0  # X=switch number
 no shutdown
```

## ✅ Verification Tests
```
# Core Functionality
show vtp status          # Revision sync
show interfaces trunk    # Native VLAN 99
show spanning-tree summary  # rapid-pvst

# Load-Balancing
show spanning-tree vlan 100 brief  # Root=SW1 (10.0.1.x)
show spanning-tree vlan 300 brief  # Root=SW4 (10.0.3.x)

# Connectivity
ping 192.168.99.1        # Mgmt VLAN
ping 10.0.1.1            # VLAN 100 gateway
ping 10.0.3.10           # VLAN 100→300 inter-VLAN
```

## 📁 Configurations
```
├── config/
│   ├── SW1.txt     (192.168.99.1 - VTP Server)
│   ├── SW4.txt     (192.168.99.4 - Root 300/400)
│   ├── R1.txt      (10.0.1.1/10.0.2.1/...)
│   └── Access.txt
└── images/
    └── topology.png
```

## 🎓 Spanning Tree Analysis

**Access Layer Design** (Single gi0/0 trunk):
```
gi0/0 → Root Port (immediate forwarding post-convergence)
gi0/1-3 → Designated + PortFast (instant host access)
Result: Simplified edge, zero local loops
```

**Load-Balancing Impact**:
```
VLAN 100/200 → SW1 root → Optimal paths to R1 (10.0.1.1/10.0.2.1)
VLAN 300/400 → SW4 root → Optimal paths to R1 (10.0.3.1/10.0.4.1)
→ 100% link utilization across VLANs
```


## 🎓 Technical Analysis

### Real-World Impact & Use Cases
**Where this topology shines**:
- **Campus networks**: Distribution layer full-mesh for redundancy, access layer single-uplink for simplicity.
- **Data centers** (collapsed core): High availability with STP load-balancing.
- **Enterprise branch**: VLAN segmentation + management VLAN 99.

**Design Benefits**:
1. **Single trunk access layer**: Simplifies cabling, STP (no loops at edge).
2. **STP load-balancing**: Doubles bandwidth utilization (VLANs split across roots).
3. **Rapid convergence**: <5s failover vs 30-50s classic STP.
4. **Native VLAN 99**: Isolates management from user VLAN 1.

### Limitations & Improvements
| Limitation | Impact | Improvement |
|------------|--------|-------------|
| **STP blocking** | 50% links unused | EtherChannel (LACP) for all paths |
| **VTP single point failure** | VLANs lost if server fails | VTP Transparent + manual VLANs or MVRP |
| **Single trunk access** | Single point of failure | Dual trunks + EtherChannel |
| **No L3** | No inter-VLAN routing | L3 SVI on distribution + OSPF/EIGRP |

**Production Enhancements**:
```cisco
! EtherChannel instead of single trunks
interface port-channel 1
 switchport mode trunk
interface range gi0/0-1
 channel-group 1 mode active

! StackWise/VSS for distribution redundancy
! VXLAN/EVPN for DC overlay (replaces STP)
```

**Sources**: Cisco Campus Design, STP Best Practices. [community.cisco](https://community.cisco.com/t5/networking-knowledge-base/frequent-cdp-4-nvlanmismatch-or-cdp-4-native-vlan-mismatch/ta-p/3130899)

