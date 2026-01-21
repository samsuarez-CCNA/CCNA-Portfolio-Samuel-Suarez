# ✅ Variant 1 – Wrong VLAN Assignment (Fixed)

## 🔧 Solution
Port **Gi0/2** was corrected to belong to **VLAN 20 (HR)**.  

```cisco
interface g0/2
 switchport mode access
 switchport access vlan 20
````

---

## 🖥️ Verification

### PC2 → Gateway Ping

```vpcs
ping 192.168.20.1
```

<img width="990" height="550" alt="fixed_vlan_assignment" src="https://github.com/user-attachments/assets/f1b6b6b9-be9e-4913-b7cd-1866e11ee529" />


Result: PC2 now successfully reaches its gateway.

---

### PC2 → PC1 Ping

```vpcs
ping 192.168.10.10
```

<img width="990" height="550" alt="successful_ping" src="https://github.com/user-attachments/assets/637be86a-5fc0-404c-a31b-1143f9749411" />


Result: PC2 and PC1 can communicate across VLANs via R1.

---

## ✅ Reflection

* Ensuring the correct VLAN assignment for switchports is **critical for connectivity**.
* After reassigning Gi0/2 to VLAN 20, both the gateway ping and inter-VLAN routing worked as expected.
* This shows why verifying configs with `show vlan brief` should always be part of troubleshooting.
