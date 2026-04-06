# 🌀 GNS3 Automation Controller & Cisco Core Setup

This guide outlines the procedure for setting up a dual-homed Ubuntu Automation Controller and establishing an SSH handshake with Cisco Core switches in a GNS3 environment.

---

## 🏗️ 1. Topology Design (Management Plane)

| Interface | Device | IP Address | Purpose |
| :--- | :--- | :--- | :--- |
| **ens3** | Ubuntu Controller | `192.168.100.10/24` | Management Station |
| **ens4** | Ubuntu Controller | `192.168.42.50/24` | Internet / NAT |
| **Gi0/0** | Cisco Core 1 | `192.168.100.11/24` | OOB Management |
| **Gi0/0** | Cisco Core 2 | `192.168.100.12/24` | OOB Management |

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
Apply with:

Bash
sudo netplan apply
🤝 3. The Cisco Handshake (IOS Configuration)
Run these commands on the Cisco Core switches to prepare for Netmiko automation.

Code snippet
! Configure Management Interface
conf t
interface GigabitEthernet0/0
 description OOB_MANAGEMENT
 ip address 192.168.100.11 255.255.255.0
 no shutdown
exit

! Security & SSH Setup
username admin privilege 15 secret [password]
ip domain-name lab.local
crypto key generate rsa general-keys modulus 2048
ip ssh version 2

! Enable SSH on VTY Lines
line vty 0 4
 login local
 transport input ssh
exit
🐍 4. Python/Netmiko Verification
Run this within your Python virtual environment (source ~/net-auto/bin/activate):

Python
from netmiko import ConnectHandler

device = {
    'device_type': 'cisco_ios',
    'host': '192.168.100.11',
    'username': 'admin',
    'password': '[password]',
}

try:
    with ConnectHandler(**device) as net_connect:
        print("✅ Handshake Successful!")
        print(net_connect.send_command('show ip int br'))
except Exception as e:
    print(f"❌ Connection Failed: {e}")
🧪 5. Troubleshooting Checklist
[ ] ARP Check: ip neighbor show should show 192.168.42.1 as REACHABLE.

[ ] Port Check: nc -zv 192.168.100.11 22 from Ubuntu should succeed.

[ ] Route Check: ip route should have exactly ONE default via 192.168.42.1.

Note: If GitHub still renders the YAML block incorrectly, ensure there is exactly one blank line before and after the triple backticks (```).

Specifications of Ubuntu VM
Ubuntu Server 24.04 image (https://www.osboxes.org/ubuntu-server/#ubuntu-server-24-04-vmware)
vCPU: 1
RAM: 512MB
Interfaces: 2

---

**Does that look cleaner on your end?** If you have any specific parts that were "exploding" (like the tables or the YAML), let me know and I can tweak the syntax!
