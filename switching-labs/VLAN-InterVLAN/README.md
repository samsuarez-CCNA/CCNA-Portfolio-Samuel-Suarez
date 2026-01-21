# 🧰 VLAN & Inter-VLAN Routing Lab

## 🎯 Overview  
This lab demonstrates how to configure **VLANs,** assign access ports, and enable **router-on-a-stick inter-VLAN routing.**
It also includes **broken variants** to practice troubleshooting, along with the **fixed solutions** — showing both diagnostic and configuration skills.  

---

## 🖥️ Topology
- 1x Cisco IOSv Router (R1)
- 1x Cisco IOSv Switch (SW1)
- 2x VPCS hosts (PC1 in VLAN 10, PC2 in VLAN 20)

_📸 See lab diagrams and configs in [LAB_GUIDE.md](LAB_GUIDE.md)
._

---

## 📂 Lab Structure
### 🔹 Base Lab
- [LAB_GUIDE.md](LAB_GUIDE.md) – full walkthrough with configs & screenshots
### 🔹 Troubleshooting Variants
- **Variant 1 – Wrong VLAN Assignment**  
  - [Broken](./variant1-broken/README.md)  
  - [Fixed](./variant1-fixed/README.md)  

- **Variant 2 – Missing Trunk**  
  - [Broken](./variant2-broken/README.md)  
  - [Fixed](./variant2-fixed/README.md)  

- **Variant 3 – Wrong Default Gateway**  
  - [Broken](./variant3-broken/README.md)  
  - [Fixed](./variant3-fixed/README.md)  

---
## 🧩 Key Skills Practiced
- VLAN creation (`show vlan brief`)
- Trunk configuration (`show interfaces trunk`)
- Router-on-a-stick subinterfaces (`encapsulation dot1Q`)
- End-device IP/gateway setup (`show ip` in VPCS)
- Troubleshooting methodology
