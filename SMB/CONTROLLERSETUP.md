# 🌀 GNS3 Automation Controller & Cisco Core Setup

This guide outlines the procedure for setting up a dual-homed Ubuntu Automation Controller and establishing an SSH handshake with Cisco Core switches in a GNS3 environment.

---

## 🏗️ 1. Topology Design (Management Plane)

| Interface | Device | IP Address | Purpose |
| :--- | :--- | :--- | :--- |
| **ens3** | Ubuntu Controller | `192.168.100.50/24` | Management Station |
| **ens4** | Ubuntu Controller | `192.168.42.50/24` | Internet / NAT |
| **Gi0/0** | Cisco Core 1 | `192.168.100.1/24` | OOB Management |
| **Gi0/0** | Cisco Core 2 | `192.168.100.2/24` | OOB Management |

---

## 🌐 2. Ubuntu Controller Configuration

### ⚡ Netplan Persistence
Edit `/etc/netplan/00-installer-config.yaml` and ensure the indentation uses **2 spaces** (no tabs):

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      addresses:
        - 192.168.100.50/24
    ens4:
      addresses:
        - 192.168.42.50/24
      routes:
        - to: default
          via: 192.168.42.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
