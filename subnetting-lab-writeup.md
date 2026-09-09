# Subnetting a /24 for a Multi-LAN Customer Network (Packet Tracer Lab Walkthrough)

*A step-by-step writeup of how I designed, configured, and tested a small customer network connected to an ISP, using a single subnet mask across four subnets carved out of 192.168.0.0/24.*

---

## The Scenario

I was given a customer site that needed to connect to an ISP over a serial WAN link. The customer side needed two LANs (LAN-A and LAN-B) plus room for future growth, all carved out of **192.168.0.0/24**. The ISP side was already fully addressed — my job was to design the customer's addressing scheme, then configure the router, switches, and PCs, and verify everything with pings.

Requirements:

- **LAN-A** needs at least **50** usable host addresses
- **LAN-B** needs at least **40** usable host addresses
- At least **2 additional subnets** reserved for future expansion
- **No VLSM** — every subnet must use the *same* mask

---

## Part 1: Designing the Subnetting Scheme

### Step 1 — Working the math

**How many host addresses are needed in the largest required subnet?**
50 (LAN-A is the bigger of the two LANs).

**What's the minimum number of subnets required?**
4 → LAN-A, LAN-B, plus 2 spares for future expansion.

**What is 192.168.0.0/24 in binary?**
```
11111111.11111111.11111111.00000000
```

**What do the 1s and 0s represent?**
- The **1s** = the **network portion** of the address (identifies the subnet)
- The **0s** = the **host portion** of the address (identifies devices on that subnet)

### Step 2 — Testing candidate masks

Borrowing bits from the host portion increases the number of subnets while shrinking the number of hosts per subnet. Here's every option from /25 to /30:

| # | Prefix | Binary (last octet) | Dotted Decimal | Subnets | Usable Hosts/Subnet |
|---|--------|----------------------|-----------------|---------|----------------------|
| 1 | /25 | `10000000` | 255.255.255.128 | 2 | 126 |
| 2 | /26 | `11000000` | 255.255.255.192 | 4 | 62 |
| 3 | /27 | `11100000` | 255.255.255.224 | 8 | 30 |
| 4 | /28 | `11110000` | 255.255.255.240 | 16 | 14 |
| 5 | /29 | `11111000` | 255.255.255.248 | 32 | 6 |
| 6 | /30 | `11111100` | 255.255.255.252 | 64 | 2 |

**Which masks meet the minimum host requirement (≥50 hosts)?**
Only **/25** (126 hosts) and **/26** (62 hosts).

**Which masks meet the minimum subnet requirement (≥4 subnets)?**
**/26** and smaller (/26, /27, /28, /29, /30) — /25 only gives 2 subnets, which isn't enough.

**Which mask satisfies *both* conditions?**
✅ **/26 — 255.255.255.192** — gives exactly 4 subnets of 62 usable hosts each. This is the sweet spot: it's the smallest block size that still clears the 50-host requirement, and it happens to produce exactly the 4 subnets needed.

### Step 3 — Deriving the subnets

With a /26 mask, each subnet block is 64 addresses wide:

| Subnet Address | Prefix | Subnet Mask |
|---|---|---|
| 192.168.0.0 | /26 | 255.255.255.192 |
| 192.168.0.64 | /26 | 255.255.255.192 |
| 192.168.0.128 | /26 | 255.255.255.192 |
| 192.168.0.192 | /26 | 255.255.255.192 |

The first two subnets get assigned to LAN-A and LAN-B. The last two are held in reserve for future expansion.

---

## Part 2: The Completed Addressing Table

Following the assignment rules — router gets the **first** usable host address, the switch gets the **second**, and the PC gets the **last** usable host address in its subnet:

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|---|
| CustomerRouter | G0/0 | 192.168.0.1 | 255.255.255.192 | N/A |
| CustomerRouter | G0/1 | 192.168.0.65 | 255.255.255.192 | N/A |
| CustomerRouter | S0/1/0 | 209.165.201.2 | 255.255.255.252 | N/A |
| LAN-A Switch | VLAN1 | 192.168.0.2 | 255.255.255.192 | 192.168.0.1 |
| LAN-B Switch | VLAN1 | 192.168.0.66 | 255.255.255.192 | 192.168.0.65 |
| PC-A | NIC | 192.168.0.62 | 255.255.255.192 | 192.168.0.1 |
| PC-B | NIC | 192.168.0.126 | 255.255.255.192 | 192.168.0.65 |
| ISPRouter | G0/0 | 209.165.200.225 | 255.255.255.224 | N/A |
| ISPRouter | S0/1/0 | 209.165.201.1 | 255.255.255.252 | N/A |
| ISPSwitch | VLAN1 | 209.165.200.226 | 255.255.255.224 | 209.165.200.225 |
| ISP Workstation | NIC | 209.165.200.235 | 255.255.255.224 | 209.165.200.225 |
| ISP Server | NIC | 209.165.200.240 | 255.255.255.224 | 209.165.200.225 |

Reserved for future growth: `192.168.0.128/26` and `192.168.0.192/26`.

---

## Part 3: Device Configuration

### CustomerRouter

```
enable
configure terminal
hostname CustomerRouter
enable secret Class123

line console 0
password Cisco123
login
exit

interface g0/0
 ip address 192.168.0.1 255.255.255.192
 no shutdown
 exit

interface g0/1
 ip address 192.168.0.65 255.255.255.192
 no shutdown
 exit

interface s0/1/0
 ip address 209.165.201.2 255.255.255.252
 no shutdown
 exit

end
copy running-config startup-config
```

### LAN-A Switch

```
enable
configure terminal
hostname LAN-A-Switch

interface vlan1
 ip address 192.168.0.2 255.255.255.192
 no shutdown
 exit

ip default-gateway 192.168.0.1
end
copy running-config startup-config
```

### LAN-B Switch

```
enable
configure terminal
hostname LAN-B-Switch

interface vlan1
 ip address 192.168.0.66 255.255.255.192
 no shutdown
 exit

ip default-gateway 192.168.0.65
end
copy running-config startup-config
```

### PC-A

| Setting | Value |
|---|---|
| IP Address | 192.168.0.62 |
| Subnet Mask | 255.255.255.192 |
| Default Gateway | 192.168.0.1 |

### PC-B

| Setting | Value |
|---|---|
| IP Address | 192.168.0.126 |
| Subnet Mask | 255.255.255.192 |
| Default Gateway | 192.168.0.65 |

---

## Part 4: Testing & Verification

With everything configured, I ran three connectivity checks:

**1. PC-A → its default gateway (192.168.0.1)**
```
C:\> ping 192.168.0.1
Reply from 192.168.0.1: bytes=32 time<1ms TTL=255
```
✅ Success.

**2. PC-B → its default gateway (192.168.0.65)**
```
C:\> ping 192.168.0.65
Reply from 192.168.0.65: bytes=32 time<1ms TTL=255
```
✅ Success.

**3. PC-A → PC-B (192.168.0.126)**
```
C:\> ping 192.168.0.126
Reply from 192.168.0.126: bytes=32 time=1ms TTL=127
```
✅ Success — traffic is being routed between the two LANs through CustomerRouter's two Gigabit interfaces.

### Troubleshooting notes

If any of these pings had failed, the checklist I'd walk through is:
1. **Double-check the mask** — every device on the customer side must use `255.255.255.192`. A mismatched mask is the #1 cause of "can't reach gateway" errors.
2. **Verify the gateway IP matches the router's interface IP** on that same subnet (PC-A/LAN-A Switch → 192.168.0.1, PC-B/LAN-B Switch → 192.168.0.65).
3. **Confirm interfaces are `no shutdown`** — a surprisingly common miss on both router and switch VLAN interfaces.
4. **Check cabling/VLAN assignment** in Packet Tracer if link lights are red.

---

## Key Takeaways

- Borrowing 2 bits from the host portion of a /24 (→ /26) is the minimum subnetting needed to satisfy *both* "≥50 hosts" and "≥4 subnets" without VLSM.
- Fixed-length subnetting is simpler to manage but less space-efficient than VLSM — here it cost us nothing since both LANs comfortably fit under 62 hosts, but it's worth remembering VLSM would let LAN-A and LAN-B use differently-sized blocks if their host counts had been very different.
- Always confirm host-portion math with 2^n − 2 (subtracting network and broadcast addresses) rather than assuming.

---

*Lab based on a standard Cisco Networking Academy subnetting/configuration exercise, completed in Packet Tracer.*
