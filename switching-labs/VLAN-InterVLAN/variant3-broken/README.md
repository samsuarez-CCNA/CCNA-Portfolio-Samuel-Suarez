# ❌ Variant 3 – Wrong Default Gateway (Broken)

## 🔎 Problem
PC2 was configured with the **wrong default gateway**.  
Instead of pointing to `192.168.20.1` (its VLAN 20 gateway), it was set to another address.  

Because of this, PC2 cannot communicate outside its subnet — including with PC1 in VLAN 10.

---

## 🖥️ Topology Snapshot
*(PC2 with incorrect default gateway)*  
<img width="990" height="550" alt="wrong_gw" src="https://github.com/user-attachments/assets/3029948f-5c83-487e-a1ba-af2e1f1f1cbc" />


---

## 🖥️ Verification

### PC2 → PC1 Ping
```vpcs
ping 192.168.10.10
````

<img width="990" height="550" alt="pc2_ping_fail" src="https://github.com/user-attachments/assets/acb5f89f-7252-43ae-ad73-a283e8097302" />


Result: The ping fails because PC2 sends traffic to a gateway that does not exist on VLAN 20.

---

## ✅ Reflection

* End devices must always use the **first-hop router IP in their subnet** as their default gateway.
* A wrong gateway setting isolates the host from all other networks.
* Always confirm device settings with `show ip` in VPCS before assuming the issue is on the router or switch.
