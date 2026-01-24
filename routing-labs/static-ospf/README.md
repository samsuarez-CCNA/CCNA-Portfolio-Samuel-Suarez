# Static vs OSPF Routing Lab

This lab demonstrates how to configure **static routes**, replace them with **OSPF dynamic routing**, and verify end-to-end connectivity between two LAN segments across three routers.  
It also contrasts **manual route configuration** with **automatic route discovery** and reconvergence.

---

## 🎯 Goals

- Build connectivity between PC1 and PC2 using **static routes**.  
- Replace static routes with **single-area OSPF**.  
- Verify neighbor adjacencies, routing tables, and pings.  
- Simulate a link failure and observe **OSPF reconvergence**.

---

## 🖥️ Topology & IP Plan

### Topology
```

PC1 --- R1 --- R2 --- R3 --- PC2

````

### IP Addressing Plan

| Device | Platform     | Interface | IP Address     | Subnet Mask       | Purpose                     |
|--------|--------------|-----------|----------------|-------------------|-----------------------------|
| **PC1** | Alpine Linux | eth0      | 192.168.10.10  | 255.255.255.0     | Host in R1 LAN              |
| **R1**  | IOSv         | Gi0/0     | 192.168.10.1   | 255.255.255.0     | Gateway for PC1 LAN         |
|        |              | Gi0/1     | 10.0.12.1      | 255.255.255.252   | P2P link to R2 (Gi1)        |
| **R2**  | CSR1000v     | Gi1       | 10.0.12.2      | 255.255.255.252   | P2P link to R1 (Gi0/1)      |
|        |              | Gi2       | 10.0.23.1      | 255.255.255.252   | P2P link to R3 (Gi0/0)      |
| **R3**  | IOSv         | Gi0/0     | 10.0.23.2      | 255.255.255.252   | P2P link to R2 (Gi2)        |
|        |              | Gi0/1     | 192.168.20.1   | 255.255.255.0     | Gateway for PC2 LAN         |
| **PC2** | Alpine Linux | eth0      | 192.168.20.10  | 255.255.255.0     | Host in R3 LAN              |

---

## Link Connections (Routing Lab)

| Link / Subnet       | Device A (Interface) | Device B (Interface) |
|---------------------|-----------------------|-----------------------|
| 192.168.10.0/24     | R1 (Gi0/0)           | PC1 (eth0)           |
| 10.0.12.0/30        | R1 (Gi0/1)           | R2 (Gi1)             |
| 10.0.23.0/30        | R2 (Gi2)             | R3 (Gi0/0)           |
| 192.168.20.0/24     | R3 (Gi0/1)           | PC2 (eth0)           |

---

## Router Interface Configurations (Baseline)

### R1
```cisco
conf t
interface g0/0
 ip address 192.168.10.1 255.255.255.0
 no shut
interface g0/1
 ip address 10.0.12.1 255.255.255.252
 no shut
end
````

### R2

```cisco
conf t
interface g1
 ip address 10.0.12.2 255.255.255.252
 no shut
interface g2
 ip address 10.0.23.1 255.255.255.252
 no shut
end
```

### R3

```cisco
conf t
interface g0/0
 ip address 10.0.23.2 255.255.255.252
 no shut
interface g0/1
 ip address 192.168.20.1 255.255.255.0
 no shut
end
```

---

## 🖥️ PC1 & PC2 Configuration (Persistent via `vi`)

Both PCs use Alpine Linux. Network settings were configured with `vi` and saved in `/etc/network/interfaces`.

### PC1

```sh
vi /etc/network/interfaces

auto eth0
iface eth0 inet static
    address 192.168.10.10
    netmask 255.255.255.0
    gateway 192.168.10.1
```

### PC2

```sh
vi /etc/network/interfaces

auto eth0
iface eth0 inet static
    address 192.168.20.10
    netmask 255.255.255.0
    gateway 192.168.20.1
```

### Apply Changes

```sh
rc-service networking restart
```

---

## 📡 Static Routing

### R1

```cisco
ip route 192.168.20.0 255.255.255.0 10.0.12.2
```

### R2

```cisco
ip route 192.168.10.0 255.255.255.0 10.0.12.1
ip route 192.168.20.0 255.255.255.0 10.0.23.2
```

### R3

```cisco
ip route 192.168.10.0 255.255.255.0 10.0.23.1
```

---

## 🌐 Connectivity Tests (Static)

From **PC1**:

```sh
ping -c 3 192.168.20.10
```

From **PC2**:

```sh
ping -c 3 192.168.10.10
traceroute 192.168.10.10
```

---

## 🛠️ Additional Static Routes

* **R1 Floating Static**

```cisco
ip route 192.168.20.0 255.255.255.0 10.0.12.2 200
```

* **R2 Default Route**

```cisco
ip route 0.0.0.0 0.0.0.0 10.0.12.1
```

---

## 🗑️ Static Routes Removed (Clean Slate)

Removed with:

```cisco
no ip route ...
```

Verification:

```cisco
show ip route
```

Only directly connected routes should remain.

Ping PC1 → PC2 should now **fail**.

---

## 🖧 OSPF Routing (Single Area)

### R1

```cisco
router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.3 area 0
```

### R2

```cisco
router ospf 1
 router-id 2.2.2.2
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.23.0 0.0.0.3 area 0
```

### R3

```cisco
router ospf 1
 router-id 3.3.3.3
 network 10.0.23.0 0.0.0.3 area 0
 network 192.168.20.0 0.0.0.255 area 0
```

---

## 🔍 Verification (OSPF)

```cisco
show ip ospf neighbor
show ip route ospf
```

Ping **PC1 → PC2** again. Should succeed.

---

## 🔻 Failure Simulation & Recovery

* Shut down R2’s Gi1 and Gi2 → neighbors drop, routes vanish.
* Restore with `no shutdown` → OSPF reconverges, routes reappear.

Verification:

```cisco
show ip ospf neighbor
show ip route ospf
```

---

## ✅ Reflection

* **Static routing** is predictable but brittle — every path must be entered manually.
* **OSPF** is dynamic, learns routes automatically, and reconverges quickly after failures.
* **Floating static + default routes** are useful as backups or at the edge.
* This lab validated both approaches with step-by-step configs, verification, and failure/recovery testing.

## 📷 Screenshots

<img width="1023" height="129" alt="topology" src="https://github.com/user-attachments/assets/f8e603f4-a755-4b32-bb6f-8da04608c902" />



<img width="2906" height="1448" alt="ip_plan" src="https://github.com/user-attachments/assets/28e52ebe-98d1-4728-8c20-5893b4f63469" />



<img width="990" height="550" alt="r1_ip_int_config" src="https://github.com/user-attachments/assets/a733ef54-6e82-4be6-b2fd-7e6bd4011731" />



<img width="990" height="550" alt="r2_ip_int_config" src="https://github.com/user-attachments/assets/fc2d7ecb-d093-4643-973f-9122fe485f8b" />



<img width="990" height="550" alt="r3_ip_int_config" src="https://github.com/user-attachments/assets/3b8accad-fdac-4e1b-877d-ab1085bb8a74" />



<img width="990" height="550" alt="pc1_ip_config" src="https://github.com/user-attachments/assets/6b318d05-8675-49ac-b58a-c1576233babb" />



<img width="990" height="550" alt="persistent_settings_pc1" src="https://github.com/user-attachments/assets/f51388ee-bc04-446a-ad66-be06da1f0009" />



<img width="990" height="550" alt="r1_static" src="https://github.com/user-attachments/assets/712bd152-7baf-4ed1-ade7-7bc13659ba49" />



<img width="990" height="550" alt="r2_static" src="https://github.com/user-attachments/assets/bed3b6ee-3070-4734-831b-7edbf1945bd3" />


<img width="990" height="550" alt="r3_static" src="https://github.com/user-attachments/assets/e2eb813f-c293-41dc-bd8e-ba68fe38e0f5" />



<img width="990" height="550" alt="pc1_to_pc2_ping" src="https://github.com/user-attachments/assets/13bcedb0-32c9-422c-b5d2-05716caace97" />



<img width="990" height="550" alt="pc2_to_pc1_ping" src="https://github.com/user-attachments/assets/a811b7b2-659f-432e-be18-6b27409f2bc3" />



<img width="990" height="210" alt="pc2_traceroute" src="https://github.com/user-attachments/assets/0a82e3a8-0632-4148-9109-68bedb3f72e1" />


<img width="990" height="550" alt="r1_static_floating" src="https://github.com/user-attachments/assets/913c07a4-92c4-442a-a9e7-d95b1263452b" />


<img width="990" height="550" alt="r2_static_default" src="https://github.com/user-attachments/assets/fecf7416-d680-40fb-a1a6-5b4d95fe3cbb" />


<img width="990" height="550" alt="R2_hostname_logging" src="https://github.com/user-attachments/assets/c21b3689-0ac8-4b9c-b1d2-571e2c512bb7" />


<img width="1260" height="849" alt="static_routes_gone" src="https://github.com/user-attachments/assets/b91a1ce0-b07a-4619-a04e-d10d4e4cbd80" />


<img width="1270" height="421" alt="ping_fail" src="https://github.com/user-attachments/assets/e8be1a4b-28b8-4f9d-997d-e4afcaa4a58d" />


<img width="990" height="550" alt="r1_ospf_config" src="https://github.com/user-attachments/assets/24e9704b-5321-4be8-bb97-6ccd89a6dc7c" />


<img width="990" height="550" alt="r2_ospf_neighbor" src="https://github.com/user-attachments/assets/d82e80b0-71b8-41a5-9043-be596900f5f8" />


<img width="1264" height="414" alt="ospf_ip_route_other_pcs" src="https://github.com/user-attachments/assets/af5c5e9f-4f17-468c-b39e-4533d0934f68" />


<img width="990" height="550" alt="pc1_ping_dynamic" src="https://github.com/user-attachments/assets/8cd856b5-777e-470a-85ce-1ca0816ab644" />


<img width="990" height="550" alt="shutdown" src="https://github.com/user-attachments/assets/cbbac3ef-d56d-4082-b6b2-7bb0f0c985c7" />


<img width="2488" height="820" alt="down_no_neighbor" src="https://github.com/user-attachments/assets/4688a722-4ff7-466f-8c35-5a5b01af4f35" />


<img width="2500" height="832" alt="routes_gone_after_shutdown" src="https://github.com/user-attachments/assets/eeb4c4e0-43bb-4546-9780-03df5fda98f3" />

<img width="2640" height="250" alt="ospf_recovery" src="https://github.com/user-attachments/assets/cd8eca56-21d5-4db0-a63d-dd8f5ec7c509" />

