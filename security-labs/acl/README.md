# 🔒 ACL Lab Report — 5 Attempts to Get It Right

## 🎯 Objectives

* Practice configuring **Standard and Extended ACLs**.
* Understand how ACL **placement (near source vs near destination)** and **direction (in vs out)** affect traffic.
* Demonstrate common mistakes and troubleshooting steps.
* Arrive at the **correct solution**: an Extended ACL applied near the source (R1 G0/1) to block **PC1 → PC2 ICMP** while allowing all else.

---

## 🖼️ Topology 
<img width="1023" height="129" alt="acl_topology" src="https://github.com/user-attachments/assets/662aa576-9677-4164-9262-fd4dbd059fb8" />

```
PC1 (192.168.10.10) ---- R1 ---- R2 ---- R3 ---- PC2 (192.168.20.10)
```

* PC1: 192.168.10.10/24 (gateway R1 G0/0)
* PC2: 192.168.20.10/24 (gateway R3 G0/1)
* Transit networks between R1–R2–R3: 10.0.x.x/30

---

# 🛠 Attempts and Outcomes

---

## 🔧 Attempt 1 (A1) — Standard ACL on R1 Inbound

```cisco
R1(config)# access-list 1 deny host 192.168.10.10
R1(config)# access-list 1 permit any
R1(config)# interface g0/0
R1(config-if)# ip access-group 1 in
```

### Result:

* PC1 → PC2 ping ❌ (blocked)
* PC1 → anywhere else ❌ (also blocked — overblocking)

📸 Screenshots:

<img width="1516" height="916" alt="show_access_a1" src="https://github.com/user-attachments/assets/40558df0-8822-45ca-88ae-d386cc964307" /> (ACL applied)
<img width="1250" height="824" alt="pc1_ping_fail_a1" src="https://github.com/user-attachments/assets/c9ac6fd4-c6c3-4e12-babb-11da97b33c81" /> (ping blocked, but too broad)

**Lesson:** Standard ACLs filter only by **source IP**, so placing them near the source blocks *all* traffic from PC1.

---

## 🔧 Attempt 2 (A2) — Standard ACL on R3 Inbound (Wrong Direction)

```cisco
R3(config)# access-list 1 deny host 192.168.10.10
R3(config)# access-list 1 permit any
R3(config)# interface g0/1
R3(config-if)# ip access-group 1 in
```

### Result:

* PC1 → PC2 ping ✅ (succeeded — ACL had no effect)

📸 Screenshots:

<img width="1606" height="1024" alt="r3_in_acl_a2" src="https://github.com/user-attachments/assets/decc5610-1a63-46e1-9abf-9263e3be3df1" /> (ACL applied)
<img width="1240" height="836" alt="pc1_ping_success_a2" src="https://github.com/user-attachments/assets/d14857c1-ab93-438a-a727-8c0012a96cc0" /> (ping succeeded, showing ACL didn’t catch traffic)

**Lesson:** PC1’s traffic doesn’t *enter* R3 on G0/1 inbound, it *exits* there. Wrong direction = ACL not triggered.

---

## 🔧 Attempt 3 (A3) — Standard ACL on R3 Outbound

```cisco
R3(config)# interface g0/1
R3(config-if)# no ip access-group 1 in
R3(config-if)# ip access-group 1 out
```

### Result:

* PC1 → PC2 ping ❌ (blocked, correct)
* PC2 → PC1 ping ❌ (failed — replies dropped too)

📸 Screenshots:

<img width="1260" height="846" alt="pc2_ping_fail_a3" src="https://github.com/user-attachments/assets/d500b31b-bebf-47c5-9ddb-d8bd785840d9" /> (PC2 ping fails)
<img width="1242" height="830" alt="pc1_ping_fail_a3" src="https://github.com/user-attachments/assets/6f3ec912-798c-476e-9e81-3affbf382d52" /> (PC1 ping fails)

**Lesson:** Standard ACLs are **stateless** and filter by source only. The echo replies from PC1 (source = 192.168.10.10) also matched the deny, breaking PC2 → PC1.

---

## 🔧 Attempt 4 (A4) — Extended ACL on R3 G0/1 Outbound

```cisco
R3(config)# ip access-list extended BLOCK-PC1-PC2
R3(config-ext-nacl)# deny ip host 192.168.10.10 host 192.168.20.10
R3(config-ext-nacl)# permit ip any any
R3(config)# interface g0/1
R3(config-if)# ip access-group BLOCK-PC1-PC2 out
```

### Result:

* PC1 → PC2 ping ❌ (blocked, correct)
* PC2 → PC1 ping ❌ (failed again — replies still blocked)

📸 Screenshot:

<img width="1244" height="842" alt="pc2_ping_fail_a4" src="https://github.com/user-attachments/assets/0bddf40c-d485-4655-bc71-d1ac3d49a0c5" />


**Lesson:** Even with Extended ACLs, using a broad `deny ip` rule blocks replies as well, since they have the same source/destination pair (PC1 ↔ PC2).

---

## 🔧 Attempt 5 (A5) — Extended ACL on R1 G0/1 Outbound (Final Correct Solution)

```cisco
R1(config)# ip access-list extended BLOCK-PC1-PC2-ICMP
R1(config-ext-nacl)# deny icmp host 192.168.10.10 host 192.168.20.10 echo
R1(config-ext-nacl)# permit ip any any
R1(config)# interface g0/1
R1(config-if)# ip access-group BLOCK-PC1-PC2-ICMP out
```

### Result:

* PC1 → PC2 ping ❌ (blocked)
* PC2 → PC1 ping ✅ (works — replies not blocked)
* PC1 → other destinations ✅ (allowed)

📸 Screenshots:

<img width="1530" height="370" alt="r1_icmp_acl_a5" src="https://github.com/user-attachments/assets/fc89995a-1d9d-4dad-902c-cb3fd1a8d5a4" /> (ACL applied)
<img width="1258" height="830" alt="pc2_ping_success_a5" src="https://github.com/user-attachments/assets/752fbe1b-7c0b-4076-ba44-184b9025b709" /> (PC2 ping succeeds, final validation)

**Lesson:**

* Extended ACLs allow precision filtering (here, only ICMP echo requests).
* Placing them **close to the source** ensures only the unwanted traffic is dropped, while replies and other flows remain intact.
* Using `echo` instead of `ip` prevents return traffic from being blocked.

---

# 🔍 Verification Commands

* `show access-lists` → verify deny counters increase only for PC1’s pings to PC2.
* `show ip interface g0/1` → confirm ACL applied outbound.
* Ping tests from PC1 and PC2 validate traffic flow.

---

# ✅ Final Takeaways

1. **A1:** Standard ACL near source = overblocking.
2. **A2:** Standard ACL wrong direction = no effect.
3. **A3:** Standard ACL outbound = blocked both directions (stateless issue).
4. **A4:** Extended ACL outbound on R3 = too broad, replies blocked.
5. **A5:** Extended ACL outbound on R1 G0/1, protocol-specific = correct solution.

**Key Exam/Real-World Insight:**

* Standard ACLs → simple source filters, place near destination.
* Extended ACLs → precise (source, destination, protocol), place near source.
* Always test both directions because ACLs are stateless.
* Narrow your deny statements (`icmp echo` vs `ip`) to avoid unintended blocks.
