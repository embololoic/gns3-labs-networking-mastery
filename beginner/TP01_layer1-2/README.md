
## TP01 

```markdown
# TP01 — VLAN Segmentation + Inter-VLAN Routing (RoAS) + Apache Server (Debian)

## 1) Scope
Build a small enterprise-style campus edge:
- 3 user VLANs (200/300/400) on an L2 switch
- Management VLAN 99 (device management)
- Router-on-a-Stick (R1) for inter-VLAN routing over an 802.1Q trunk
- Debian server running Apache behind R1
- Intentional misconfiguration scenario: PC in VLAN200 cannot reach other VLANs because the switchport is NOT configured as an access port

<img width="1305" height="696" alt="Capture d&#39;écran 2026-02-16 104951" src="https://github.com/user-attachments/assets/f92b3aa2-a9d2-4141-a23a-538151c414d5" />

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

Links:
- SW Gi0/0 <-> R1 Gi4/0 (802.1Q trunk)
- R1 Fa0/0 <-> SERVER ens4 (L3 point-to-point)
- PCs connected to SW access ports

## 4) IP Plan
User VLANs:
- VLAN 200: 10.0.2.0/24  (GW: 10.0.2.1)
- VLAN 300: 10.0.3.0/24  (GW: 10.0.3.1)
- VLAN 400: 10.0.4.0/24  (GW: 10.0.4.1)

Management VLAN:
- VLAN 99: 192.168.99.0/24 (GW: 192.168.99.1)
- SW SVI: 192.168.99.10/24

Server link:
- R1 <-> SERVER: 10.0.99.0/30
  - R1: 10.0.99.1/30
  - SERVER: 10.0.99.2/30

Host IPs (example):
- PC-VLAN200: 10.0.2.10/24, GW 10.0.2.1
- PC-VLAN300: 10.0.3.10/24, GW 10.0.3.1
- PC-VLAN400: 10.0.4.10/24, GW 10.0.4.1
```

## 5) Intentional Failure Scenario (Designed Outage)

### 5.1 Change request (intentional)
Goal: demonstrate troubleshooting skills by creating a controlled outage.

Misconfiguration: **PC-VLAN200 switchport not configured as an access port in VLAN 200** (left in dynamic/auto/default VLAN 1).

### 5.2 Expected symptoms
From PC2-VLAN200 (10.0.2.11):
- `ping 10.0.2.10` fails
- <img width="742" height="311" alt="image" src="https://github.com/user-attachments/assets/4916c30a-4cf1-4647-b6d2-c2a406fbf23e" />
- `ping 10.0.3.10` fails
-   <img width="399" height="87" alt="image" src="https://github.com/user-attachments/assets/8a34a22e-cdca-4a5b-9330-5de5ba3b9b43" />
- `ping 10.0.4.10` fails
- <img width="425" height="82" alt="image" src="https://github.com/user-attachments/assets/42964f47-d9c8-482c-8be1-4dbce7a5c05e" />
- `ping 10.0.2.1` (gateway) may fail depending on the exact port state/VLAN membership
- <img width="430" height="92" alt="image" src="https://github.com/user-attachments/assets/4028a128-2e1b-468a-bb84-b0374fa5d0f8" />
- ARP table incomplete or incorrect

### 5.3 Evidence to collect (for the report)
On SW:
```ios
show vlan brief
show interfaces gi0/1 switchport     ! replace with the port where PC-VLAN200 is connected
show mac address-table vlan 200
```

On PC-VLAN200:
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

### 6.4 Root cause (analysis)
The PC port is not in VLAN 200 (no `switchport mode access` + `switchport access vlan 200`), so frames from the PC are placed into the wrong VLAN (often VLAN 1). Inter-VLAN routing cannot work because the host is not actually in the expected L2 broadcast domain.

### 6.5 Fix (change implementation)
```ios
conf t
interface gi0/1              ! replace with PC-VLAN200 port
 switchport mode access
 switchport access vlan 200
 spanning-tree portfast
 no shutdown
end
```

### 6.6 Post-fix validation
From PC-VLAN200:
```bash
ping -c 3 10.0.2.1
ping -c 3 10.0.3.10
ping -c 3 10.0.4.10
curl -I http://10.0.99.2
```

## 7) Acceptance Tests (Definition of Done)
- SW trunk to R1 is up and carries VLANs 99/200/300/400
- Each PC can ping its default gateway
- Inter-VLAN ping works (200 <-> 300 <-> 400)
- All PCs can reach Apache on `http://10.0.99.2`
- Evidence captured: show commands + at least 2 Wireshark screenshots (failure + after fix)

## 8) GitHub Deliverables
Commit:
- `README.md` (this document)
- `configs/` (switch + router configs as text)
- `screenshots/` (topology + key outputs)
- `captures/` (optional: pcapng files, or at least screenshots)

Suggested commit messages:
- `TP01: initial VLANs + RoAS + Apache`
- `TP01: intentional outage (access port missing) + troubleshooting`
- `TP01: fix access port + validation`
```

## Notes (important corrections)

- The reason your earlier trunk command failed is typical on platforms that require forcing `switchport trunk encapsulation dot1q` before `switchport mode trunk`. [community.cisco](https://community.cisco.com/t5/other-network-architecture-subjects/not-able-to-change-auto-to-trunk-mode-on-switch/td-p/537664)
- Router-on-a-stick subinterfaces use `encapsulation dot1q <vlan> [native]` and the subinterface IP typically serves as the VLAN default gateway. [ciscopress](https://www.ciscopress.com/articles/article.asp?p=3089357&seqNum=5)
- On Cisco switches, an access port should be explicitly set with `switchport mode access` and `switchport access vlan <id>`. [connecteddots](https://www.connecteddots.online/resources/cisco-reference/switchport-access-vlan)
- On Debian, Apache is managed as a systemd service (`systemctl start/status/enable apache2`). [reintech](https://reintech.io/blog/installing-apache-on-debian-12-step-by-step-guide)

## Quick question (so I match your lab exactly)
Which switch/router images are you using in GNS3 (IOSvL2 + IOSv, IOL, etc.) and what are the exact interface names for the PC ports (Gi0/1, Gi0/2, Gi0/3)?
