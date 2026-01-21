# ✅ Variant 3 – Wrong Default Gateway (Fixed)

## 🔧 Solution
PC2’s default gateway was corrected to `192.168.20.1`, which is the router subinterface for VLAN 20.  

```vpcs
ip 192.168.20.10 255.255.255.0 192.168.20.1
show ip
````

---

## 🖥️ Verification

### PC2 → Gateway Check

<img width="990" height="550" alt="correct_gw_assignment" src="https://github.com/user-attachments/assets/d44d9d9b-0991-4f40-87c2-8842186a7fc5" />


---

### PC2 → PC1 Ping

```vpcs
ping 192.168.10.10
```

<img width="990" height="550" alt="good_ping" src="https://github.com/user-attachments/assets/42f1f3f3-f0ef-4725-94a9-0d8ac429b401" />


Result: Connectivity between PC2 and PC1 is restored once the gateway is corrected.

---

## ✅ Reflection

* The **default gateway must match the router’s IP in the same VLAN subnet**.
* With the proper gateway set, PC2 can route traffic to other VLANs.
* Verifying with `show ip` on VPCS ensures correct addressing before deeper troubleshooting.
