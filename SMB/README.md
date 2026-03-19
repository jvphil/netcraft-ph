# 🌐 SMB Proposed Network Topology

> A fully redundant, enterprise-grade network topology designed for Small-to-Medium Businesses (SMB) — built and simulated using Cisco Packet Tracer.

**Author:** [@jvphil](https://github.com/jvphil)

---

## 📋 Overview

This project demonstrates a compressed core network architecture with full redundancy at every layer — from dual ISP links down to enterprise wireless access points. Designed with cost-efficiency in mind using open-source firewall software (pfSense) and budget-friendly enterprise hardware.

---

## 🗺️ Topology Diagram

```
        ISP 1                    ISP 2
    10.254.233.1              10.254.223.1
         |                         |
        FW1                       FW2
   (pfSense/ASA)             (pfSense/ASA)
   10.254.233.2              10.254.223.2
   200.10.208.1              129.12.108.1
         |                         |
        SW1 =================== SW2
   (Core L3 Switch)         (Core L3 Switch)
   200.10.208.2              129.12.108.2
   192.168.1.2               192.168.1.3
   HSRP Active               HSRP Standby
   STP Primary               STP Secondary
   DHCP .11-.128             DHCP .129-.254
         |                         |
        AP1                       AP2
   (Enterprise AP)           (Enterprise AP)
```

---

## 📡 Technologies Used

| Technology | Purpose |
|---|---|
| **HSRP** | Gateway redundancy — Virtual IP `192.168.1.1` |
| **STP** | Loop prevention, root bridge election |
| **EtherChannel (LACP)** | Link aggregation between core switches |
| **NAT (Dynamic)** | Inside-to-outside address translation |
| **Static Routing** | Inter-device routing |
| **DHCP** | Split pool — redundant address assignment |
| **ICMP Inspection** | Firewall policy for ICMP traffic |
| **VLAN 10** | User/data VLAN (VLAN 1 disabled per best practice) |

---

## 🗂️ IP Addressing Table

| Device | Interface | IP Address | Subnet | Notes |
|---|---|---|---|---|
| ISP1 | G0/0 | 10.254.233.1 | /30 | ISP1 gateway |
| ISP2 | G0/0 | 10.254.223.1 | /30 | ISP2 gateway |
| FW1 | Outside (G1/8) | 10.254.233.2 | /30 | FW1 outside |
| FW1 | Inside (G1/1) | 200.10.208.1 | /29 | FW1 inside |
| FW2 | Outside (G1/8) | 10.254.223.2 | /30 | FW2 outside |
| FW2 | Inside (G1/1) | 129.12.108.1 | /29 | FW2 inside |
| SW1 | To FW1 | 200.10.208.2 | /29 | Uplink to FW1 |
| SW1 | VLAN 10 | 192.168.1.2 | /24 | HSRP Active |
| SW2 | To FW2 | 129.12.108.2 | /29 | Uplink to FW2 |
| SW2 | VLAN 10 | 192.168.1.3 | /24 | HSRP Standby |
| HSRP VIP | VLAN 10 | 192.168.1.1 | /24 | Default Gateway |

---

## ⚙️ Key Configurations

### HSRP (SW1 — Active)
```
interface vlan 10
 ip address 192.168.1.2 255.255.255.0
 standby 1 ip 192.168.1.1
 standby 1 priority 110
 standby 1 preempt
 no shutdown
```

### HSRP (SW2 — Standby)
```
interface vlan 10
 ip address 192.168.1.3 255.255.255.0
 standby 1 ip 192.168.1.1
 standby 1 priority 90
 standby 1 preempt
 no shutdown
```

### EtherChannel (LACP)
```
interface range f0/1-2
 channel-group 10 mode active
!
interface port-channel 10
 switchport mode trunk
 switchport trunk allowed vlan all
```

### STP Root Bridge
```
! SW1
spanning-tree vlan 10 root primary

! SW2
spanning-tree vlan 10 root secondary
```

### Split DHCP Pools
```
! SW1 — serves .11 to .128
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp excluded-address 192.168.1.129 192.168.1.254
ip dhcp pool VLAN10
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8

! SW2 — serves .129 to .254
ip dhcp excluded-address 192.168.1.1 192.168.1.128
ip dhcp pool VLAN10
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
```

### Firewall NAT + ICMP Inspection (FW1 & FW2)
```
object network INSIDE-NET
 subnet 192.168.1.0 255.255.255.0
 nat (inside,outside) dynamic interface
!
policy-map global_policy
 class inspection_default
  inspect icmp
```

### Firewall Static Routes
```
! FW1
route outside 0.0.0.0 0.0.0.0 10.254.233.1 1
route inside 192.168.1.0 255.255.255.0 200.10.208.2 1

! FW2
route outside 0.0.0.0 0.0.0.0 10.254.223.1 1
route inside 192.168.1.0 255.255.255.0 129.12.108.2 1
```

> ⚠️ **Firewall Disclaimer:** The firewall configurations in this repository were created and tested using an **emulated Cisco ASA in Cisco Packet Tracer**. Actual firewall commands and configurations may differ depending on the firewall platform, vendor, firmware version, and deployment environment. If you are deploying this topology in a production environment using pfSense, Fortinet, Palo Alto, or any other firewall solution, **please consult your firewall's official documentation** before applying any configurations. The author assumes no responsibility for misconfigurations in production environments.

---

## 💰 Estimated Cost (Philippine Peso)

> Based on Philippine market prices (2025–2026). pfSense is free/open-source.

| Component | Specs | Qty | Unit Price | Total |
|---|---|---|---|---|
| **Beelink Mini S12** (pfSense host) | Intel N95, 8GB RAM, 256GB SSD | 2 | ₱10,995 | ₱21,990 |
| **Cisco Catalyst 3560** (refurbished) | 24-port, Layer 3, Cisco IOS | 2 | ~₱4,000 | ₱8,000 |
| **Ubiquiti U6 Pro** (Enterprise AP) | WiFi 6, 300+ clients | 2 | ~₱14,000 | ₱28,000 |
| **CAT6 Cables & misc** | — | — | ₱2,000 | ₱2,000 |
| **pfSense OS** | Open-source | 2 | ₱0 | ₱0 |
| **TOTAL** | | | | **~₱59,990** |

> 💡 Compared to a full Cisco stack (~₱300,000+), this topology delivers equivalent redundancy at ~**1/5 the cost**.
> 
> ⚠️ **Note:** Refurbished Cisco Catalyst 3560 switches are used to ensure full Cisco IOS CLI compatibility with the configurations in this repository.

> 🔒 **Security Disclaimer:** The Cisco Catalyst 3560 is End-of-Life (EOL) and no longer receives security patches from Cisco. However, in this topology, the switches sit **behind dual pfSense firewalls** and are never directly exposed to the internet. The security risk is significantly mitigated provided that:
> - pfSense firewalls are **kept up-to-date** with the latest security patches and firmware
> - Firewall rules are **properly configured and regularly reviewed**
> - Network access to the switches is **restricted to authorized administrators only**
>
> This topology is **not recommended for high-security environments** (e.g., finance, healthcare, government). For such deployments, replace the Catalyst 3560 with an actively supported switch such as the **Cisco Catalyst 9200** or equivalent.

---

## ✅ Redundancy Summary

| Layer | Redundancy Mechanism |
|---|---|
| ISP | Dual ISP links |
| Firewall | Dual pfSense firewalls |
| Core Switch | Dual L3 switches + EtherChannel |
| Gateway | HSRP (Active/Standby) |
| Spanning Tree | STP Primary/Secondary |
| DHCP | Split pools across both switches |
| Wireless | Dual APs, same SSID (client roaming) |

---

## 🛠️ Tools Used

- **Cisco Packet Tracer** — Network simulation
- **pfSense / Cisco ASA** — Firewall (pfSense on Beelink Mini S12 for production)
- **Cisco Catalyst 3560** (refurbished) — Layer 3 core switching
- **Ubiquiti U6 Pro** — Enterprise wireless

---

## 📁 Repository Structure

```
smb-network-topology/
├── README.md
├── topology/
│   └── smb_topology.pkt
├── configs/
│   ├── SW1.txt
│   ├── SW2.txt
│   ├── FW1.txt
│   └── FW2.txt
└── docs/
    ├── ip_addressing.md
    └── cost_benefit.md
```

---

## 📜 License

MIT License — feel free to use, modify, and share!

---

> *Built with 💪 and a lot of brain clouds. 🐟*
