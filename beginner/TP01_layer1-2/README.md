# TP01 — VLAN Segmentation + Inter-VLAN Routing (RoAS) + Apache Server (Debian)

## 1) Scope
Build a small enterprise-style campus edge:
- 3 user VLANs (200/300/400) on an L2 switch
- Management VLAN 99 (device management)
- Router-on-a-Stick (R1) for inter-VLAN routing over an 802.1Q trunk
- Debian server running Apache behind R1
- Intentional misconfiguration scenario: PC in VLAN200 cannot reach other VLANs because the switchport is NOT configured as an access port

## 2) Prerequisites
### Skills
- VLANs (access vs trunk), 802.1Q tagging, native VLAN concept
- Basic Cisco IOS CLI navigation and show commands
- Debian networking with `ip` command

### Tools
- GNS3 (local or VM)
- Wireshark (on Debian PC or on GNS3 link capture)

### Images / Compliance
- Public repo: only configs, documentation, screenshots, and free OS images.
- Proprietary images (Cisco/Fortinet) stay local and are NOT committed.

## 3) Topology
Devices:
- SW: L2 switch (Cisco-like)
- R1: router
- PC-VLAN200: Debian
- PC-VLAN300: Debian
- PC-VLAN400: Debian
- SERVER: Debian (Apache)

![Topology](https://github.com/user-attachments/assets/f92b3aa2-a9d2-4141-a23a-538151c414d5)

Links:
- SW Gi0/0 <-> R1 Gi4/0 (802.1Q trunk)
- R1 Fa0/0 <-> SERVER ens4 (L3 point-to-point)
- PCs connected to SW access ports

## 4) IP Plan

### User VLANs
- VLAN 200: 10.0.2.0/24  (GW: 10.0.2.1)
- VLAN 300: 10.0.3.0/24  (GW: 10.0.3.1)
- VLAN 400: 10.0.4.0/24  (GW: 10.0.4.1)

### Management VLAN
- VLAN 99: 192.168.99.0/24 (GW: 192.168.99.1)
- SW SVI: 192.168.99.10/24

### Server link
- R1 <-> SERVER: 10.0.99.0/30  
  - R1: 10.0.99.1/30  
  - SERVER: 10.0.99.2/30  

### Host IPs (example)
- PC-VLAN200: 10.0.2.10/24, GW 10.0.2.1
- PC-VLAN300: 10.0.3.10/24, GW 10.0.3.1
- PC-VLAN400: 10.0.4.10/24, GW 10.0.4.1

---

## 5) Intentional Failure Scenario (Designed Outage)

### 5.1 Change request (intentional)
Goal: demonstrate troubleshooting skills by creating a controlled outage.

Misconfiguration: **PC-VLAN200 switchport not configured as an access port in VLAN 200** (left in dynamic/auto/default VLAN 1).

---

### 5.2 Expected symptoms
From PC2-VLAN200 (10.0.2.11):

- `ping 10.0.2.10` fails  
  ![](https://github.com/user-attachments/assets/4916c30a-4cf1-4647-b6d2-c2a406fbf23e)

- `ping 10.0.3.10` fails  
  ![](https://github.com/user-attachments/assets/8a34a22e-cdca-4a5b-9330-5de5ba3b9b43)

- `ping 10.0.4.10` fails  
  ![](https://github.com/user-attachments/assets/42964f47-d9c8-482c-8be1-4dbce7a5c05e)

- `ping 10.0.2.1` (gateway) may fail depending on the exact port state/VLAN membership  
  ![](https://github.com/user-attachments/assets/4028a128-2e1b-468a-bb84-b0374fa5d0f8)

- ARP table incomplete or incorrect

---

### 5.3 Evidence to collect (for the report)

On SW:

```ios
show vlan brief
show interfaces gi0/1 switchport
show mac address-table vlan 200
```

On PC2-VLAN200:

```bash
ip addr show ens4
ip route
ping -c 3 10.0.2.1
arp -n
```

Wireshark / tcpdump on PC-VLAN200:

```bash
sudo tcpdump -eni ens4 arp or icmp
```

What you should see:
- ARP requests leaving the host
- No ARP replies for the expected gateway (if VLAN mismatch exists)
- No routed traffic to other VLANs

---

### 5.4 Root cause (analysis)

The PC port is not in VLAN 200 (no `switchport mode access` + `switchport access vlan 200`), so frames from the PC are placed into the wrong VLAN (often VLAN 1). Inter-VLAN routing cannot work because the host is not actually in the expected L2 broadcast domain.

---

### 5.5 Fix (change implementation)

```ios
conf t
interface gi0/1
 switchport mode access
 switchport access vlan 200
 spanning-tree portfast
 no shutdown
end
```

---

### 5.6 Post-fix validation

From PC2-VLAN200:

```bash
ping -c 3 10.0.2.10
```

![](https://github.com/user-attachments/assets/709f55e6-3991-4539-a606-d4fb79abb54a)

```bash
ping -c 3 10.0.2.1
```

![](https://github.com/user-attachments/assets/f256cc9b-8d51-4ebf-b6fd-06d2dbc3af5c)  
![](https://github.com/user-attachments/assets/f9d165e0-b6cd-4a7f-80ce-7ea074871d93)

---

## 7) Acceptance Tests (Definition of Done)

- SW trunk to R1 is up and carries VLANs 99/200/300/400
- Each PC can ping its default gateway
- Inter-VLAN ping works (200 <-> 300 <-> 400)
- All PCs can reach Apache on `http://10.0.99.2`
- Evidence captured: show commands + at least 2 Wireshark screenshots (failure + after fix)
