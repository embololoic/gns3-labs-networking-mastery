# gns3-labs-networking-mastery
OSI Network Labs on GNS3 + Wireshark : Series of practical labs on the OSI model, from the basics (L1-L2) to application protocols (L7).   Each lab includes GNS3 topology, Wireshark captures, and analysis.


# 🌐 Networking Mastery Labs - **CCNA → CCNP → CCIE Enterprise**

[![GitHub stars](https://img.shields.io/github/stars/embololoic/gns3-labs-networking-mastery)](https://github.com/embololoic/gns3-labs-networking-mastery)

## 🎖️ **40+ Labs **
**SD-WAN • BGP • MPLS • EVPN • Automation**



## 📋 **prerequisites**
🟢 **Beginner** : GNS3 + Wireshark  
🟡 **CCNP** : IOSv 15.9 + vEdge images  
🔴 **CCIE** : GNS3 Pro + 32GB RAM  


## 📁 **Complete Project Directory Structure** 
```
gns3-labs-networking-mastery/
|
|-- README.md                           # Portfolio landing page
|-- ROADMAP.md                          # CCNA→CCNP→CCIE progression
|-- LICENSE                             # MIT License (auto-generated)
|-- .gitignore                          # Python template (auto)
|
|-- docs/                               # Documentation
|   |-- installation.md                 # GNS3 setup + image sources
|   |-- prerequisites.md                # Hardware/software reqs
|   `-- cheatsheets/                    # OSI, BGP, Wireshark filters
|
|-- common/                             # Reusable assets
|   |-- gns3-templates/                 # .gns3proj base files
|   |-- startup-configs/                # IOS configs
|   |   |-- router-base.cfg
|   |   `-- switch-base.cfg
|   `-- wireshark-filters/              # .ghs filter files
|
|-- beginner/                           # CCNA Level (10 Labs)
|   |-- TP01_layer1-2/
|   |   |-- TP01.gns3project/
|   |   |-- screenshots/
|   |   |-- captures/
|   |   `-- README.md
|   |-- TP02_layer3-ip/
|   |-- TP05_vlan-stp/
|   `-- TP10_ospf-nat/
|
|-- intermediate/                       # CCNP ENCOR/ENARSI (20 Labs)
|   |-- TP11_bgp-basics/
|   |-- TP14_mpls-l3vpn/
|   |-- TP17_dmvpn-phase3/
|   |-- TP20_sdwan-vmanage/
|   `-- TP25_netconf-automation/
|
|-- enterprise/                         # CCIE Enterprise Infra (15 Labs)
|   |-- TP30_sdwan-advanced/
|   |-- TP33_mpls-te/
|   |-- TP36_evpn-vxlan/
|   `-- TP40_ansible-ztp/
|
`-- scenarios/                          # Production Scenarios
    |-- scenario01_datacenter-ha/
    |-- scenario02_mpls-migration/
    `-- scenario03_sdwan-hybrid-cloud/
```
