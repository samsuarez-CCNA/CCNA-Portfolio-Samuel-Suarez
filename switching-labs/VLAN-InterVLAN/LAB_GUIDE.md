# VLAN & Inter-VLAN Routing Lab
This lab demonstrates how to configure **VLANs**, assign access ports, set up a **router-on-a-stick**, and verify inter-VLAN communication using VPCS hosts.
It also links to **troubleshooting variants** where small misconfigurations (wrong VLAN, missing trunk, wrong default gateway) are introduced intentionally. These showcase both **diagnostic process** and **fixes**, simulating real-world troubleshooting.
## 🎯 Goal
- Create VLANs and assign ports on the switch.
- Configure a trunk toward the router.
- Build router subinterfaces for Inter-VLAN routing.
- Assign IPs to VPCS hosts and verify connectivity.
---
## 🖥️ Topology Snapshot
_(screenshot this from GNS3 canvas, before configs – Step 1) _
<img width="983" height="440" alt="topology" src="https://github.com/user-attachments/assets/7eb5753b-1912-4707-8359-fd455a69d73c" />  

---
## 🔧 Switch Configuration (SW1)
### 1. Create VLANs  
```bash
vlan 10
 name SALES
vlan 20
 name HR
```
<img width="990" height="550" alt="vlan_creation" src="https://github.com/user-attachments/assets/9ba2fb71-029e-4133-a078-48f165b5b59d" />

---
### 2. Assign Access Ports
```bash
interface g0/1
 switchport mode access
 switchport access vlan 10

interface g0/2
 switchport mode access
 switchport access vlan 20
```

<img width="999" height="550" alt="vlan_port_assignment" src="https://github.com/user-attachments/assets/95b52816-13dc-4ced-9e3b-883dd4175847" />

### 3. Configure Trunk Port (to R1)

```bash
interface g0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
```
<img width="999" height="550" alt="trunk_link_creation" src="https://github.com/user-attachments/assets/a509f443-d073-4e06-b2b9-50e4d1842f89" />

---
### 4. Save Config
```bash
end
write memory
```
---
## 🚫 Skipping the Initial Configuration Dialog
When the switch or router boots:
```bash
Would you like to enter the initial configuration dialog? [yes/no]:
```
👉 Type `no`. This avoids auto-generated configs and enforces manual, professional setup.

<img width="999" height="550" alt="skip_initial_config" src="https://github.com/user-attachments/assets/62f5d7dc-237b-40ce-bd40-413ca4c3b226" />

---
## 🖧 Router Configuration (R1 – Router-on-a-Stick)
```bash
hostname R1

interface g0/0
 no shutdown

interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown

end
write memory
```
<img width="999" height="550" alt="router_interfaces" src="https://github.com/user-attachments/assets/1b900d9c-11ad-438c-89c9-ca35390dbc00" />

---
## 🖥️ End Device Configuration
**PC1 (VPCS – VLAN 10)**
```bash
ip 192.168.10.10 255.255.255.0 192.168.10.1
show ip
```
<img width="999" height="550" alt="ip_output_pc1" src="https://github.com/user-attachments/assets/af7b6708-7725-4edd-ae33-049e4c593cbc" />

---
**PC2 (VPCS – VLAN 20)**
```bash
ip 192.168.20.10 255.255.255.0 192.168.20.1
show ip
```
<img width="999" height="550" alt="ip_output_pc2" src="https://github.com/user-attachments/assets/0821a443-a1a9-4cd5-8afe-7d6c56438127" />

---
## 🌐 Connectivity Tests
**From PC1 → PC2**
```bash
ping 192.168.20.10
```
<img width="999" height="550" alt="pc1_ping" src="https://github.com/user-attachments/assets/87c34d21-d06c-419a-9f77-ecc9f9f0a4b0" />

---
**From PC2 → PC1**
```bash
ping 192.168.10.10
```
<img width="999" height="550" alt="pc2_ping" src="https://github.com/user-attachments/assets/91ea16a3-8f3f-40f7-b262-88f8c949186d" />

---
## ✅ Reflection
- VLANs successfully segmented traffic at Layer 2.
- Router-on-a-stick allowed routing between VLAN 10 and VLAN 20.
- End devices could ping across VLANs once correct gateways were configured.
- This validates: VLAN creation, port assignment, trunking, router subinterfaces, and host configs.

---
## 🔀 Troubleshooting Variants
This lab also includes intentionally broken scenarios to showcase troubleshooting and repair skills.
### Variant 1 – Wrong VLAN Assignment
- Broken Config
- Fixed Config
### Variant 2 – Missing Trunk
- Broken Config
- Fixed Config
### Variant 3 – Wrong Default Gateway
- Broken Config
- Fixed Config

---
✅ These demonstrate how small misconfigurations can break connectivity — and how to systematically troubleshoot using show, ping, and config verificatio

---
## 📄 Raw Config Files
For quick reference, the full device configs are available as plain text:
- [switch-config.txt](switching-labs/VLAN-InterVLAN/router-config.txt) – VLANs, access ports, and trunk config
- [router-config.txt](VLAN-InterVLAN/router-config.txt) – Router-on-a-stick subinterfaces
- [vpcs-config.txt](VLAN-InterVLAN/vpcs-config.txt) – IP/gateway setup for PC1 & PC2
  
👉 These files are clean exports of the exact commands used in this lab, making it easy to review or reuse in future projects.
