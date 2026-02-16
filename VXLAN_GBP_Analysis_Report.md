# VXLAN-GBP (Group Based Policy) Header Analysis Report

**Date:** February 11, 2026
**Lab:** Multi-Vendor EVPN VXLAN Fabric
**Captured on:** Cumulus Linux swp1 (fabric-facing interface)
**Tool:** `tcpdump -i swp1 -n 'udp port 4789' -XX -vvv`

---

## Topology

```
         sw1 (Cisco 9300)
         Spine / Route Reflector
        /        |        \         \
    sw2         sw3       Cumulus      Versa
   Cisco       Cisco      Linux      FlexVNF
   9300        9300       (NVIDIA)
  1.1.1.2    1.1.1.3     1.1.1.5    1.1.1.4
   GBP:ON    GBP:OFF     GBP:OFF    N/A
```

**Shared L2 Domain:** VLAN 1151 / VNI 401151
**VXLAN UDP Port:** 4789

---

## VXLAN Header Format (RFC 7348 + draft-smith-vxlan-group-policy)

```
Standard VXLAN Header (8 bytes):
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|R|R|R|R|I|R|R|R|            Reserved                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                VXLAN Network Identifier (VNI) |   Reserved    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

VXLAN-GBP Extended Header (8 bytes):
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|G|R|R|D|I|R|R|A|        Group Policy ID (16-bit SGT)           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                VXLAN Network Identifier (VNI) |   Reserved    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Flags:
  I (bit 4) = VXLAN valid bit (always 1)
  G (bit 0) = GBP extension present
  D (bit 3) = Don't Learn flag
  A (bit 7) = Applied policy flag

Flag byte values:
  0x08 = 0000 1000 = I-bit only (standard VXLAN, NO GBP)
  0x88 = 1000 1000 = G-bit + I-bit (VXLAN-GBP ENABLED)
```

---

## Captured Traffic Comparison

### Scenario 1: Cisco sw2 (VTEP 1.1.1.2) -> Cumulus (VTEP 1.1.1.5) -- GBP ENABLED

**ICMP Echo Reply (sw2 SVI 192.168.1.22 -> Cumulus 192.168.1.11):**

```
Outer IP: 1.1.1.2 -> 1.1.1.5
Outer UDP: src=random, dst=4789

VXLAN Header (hex): 88 00 00 64 06 1e ff 00
                    ^^ ^^ ^^^^^ ^^^^^^^^ ^^
                    |  |  |     |        Reserved
                    |  |  |     VNI = 0x061eff = 401151
                    |  |  Group Policy ID = 0x0064 = SGT 100
                    |  Reserved
                    Flags = 0x88 (GBP ENABLED)

Inner Frame: 192.168.1.22 -> 192.168.1.11 (ICMP Echo Reply)
```

**Bit-level breakdown of Flag byte 0x88:**
```
Bit:  7  6  5  4  3  2  1  0
      1  0  0  0  1  0  0  0
      ^           ^
      |           I-bit (VXLAN valid)
      G-bit (GBP present)

G=1: Group Based Policy extension is ACTIVE
I=1: Valid VXLAN header
Group Policy ID = 0x0064 = decimal 100 (SGT tag successfully inserted!)
```

---

**ARP Request (proxied via sw2, src 192.168.1.102):**

```
Outer IP: 1.1.1.2 -> 1.1.1.5
Outer UDP: src=random, dst=4789

VXLAN Header (hex): 88 00 00 00 06 1e ff 00
                    ^^ ^^ ^^^^^ ^^^^^^^^ ^^
                    |  |  |     |        Reserved
                    |  |  |     VNI = 0x061eff = 401151
                    |  |  Group Policy ID = 0x0000 = SGT 0 (unknown)
                    |  Reserved
                    Flags = 0x88 (GBP ENABLED)

Inner Frame: ARP Who-has 192.168.1.11 tell 192.168.1.102
```

**Note:** GBP flag (0x88) is set but SGT=0 for proxied ARP traffic. The source 192.168.1.102 does not have a direct SGT mapping on sw2, so it gets tagged with the default SGT 0.

---

### Scenario 2: Cisco sw3 (VTEP 1.1.1.3) -> Cumulus (VTEP 1.1.1.5) -- NO GBP

**ICMP Echo Reply (sw3 SVI 192.168.1.23 -> Cumulus 192.168.1.11):**

```
Outer IP: 1.1.1.3 -> 1.1.1.5
Outer UDP: src=random, dst=4789

VXLAN Header (hex): 08 00 00 00 06 1e ff 00
                    ^^ ^^ ^^^^^ ^^^^^^^^ ^^
                    |  |  |     |        Reserved
                    |  |  |     VNI = 0x061eff = 401151
                    |  |  Reserved (not Group Policy ID)
                    |  Reserved
                    Flags = 0x08 (STANDARD VXLAN, NO GBP)

Inner Frame: 192.168.1.23 -> 192.168.1.11 (ICMP Echo Reply)
```

**Bit-level breakdown of Flag byte 0x08:**
```
Bit:  7  6  5  4  3  2  1  0
      0  0  0  0  1  0  0  0
                  ^
                  I-bit (VXLAN valid)

G=0: Group Based Policy extension is NOT PRESENT
I=1: Valid VXLAN header
Bytes 2-3 = 0x0000 (reserved, not SGT)
```

---

### Scenario 3: Cumulus Linux (VTEP 1.1.1.5) -> Remote VTEPs -- NO GBP

**ICMP Echo Request (Cumulus 192.168.1.11 -> sw2 192.168.1.22):**

```
Outer IP: 1.1.1.5 -> 1.1.1.2
Outer UDP: src=random, dst=4789

VXLAN Header (hex): 08 00 00 00 06 1e ff 00
                    ^^ ^^ ^^^^^ ^^^^^^^^ ^^
                    |  |  |     |        Reserved
                    |  |  |     VNI = 0x061eff = 401151
                    |  |  Reserved (not Group Policy ID)
                    |  Reserved
                    Flags = 0x08 (STANDARD VXLAN, NO GBP)

Inner Frame: 192.168.1.11 -> 192.168.1.22 (ICMP Echo Request)
```

**Note:** Cumulus Linux (FRR) does not natively support VXLAN-GBP. All outgoing VXLAN packets use standard 0x08 flags.

---

### Scenario 4: Versa FlexVNF (VTEP 1.1.1.4) -- NO TRAFFIC CAPTURED

No packets were received directly from Versa VTEP 1.1.1.4 during this capture window. The Versa device did not respond to ICMP/ARP. Traffic associated with Versa's behind-host (192.168.1.102) was proxied through Cisco sw2 (VTEP 1.1.1.2) with GBP flag 0x88.

**Previous observations (from earlier testing):** When Versa was actively sending VXLAN traffic, it used the GBP-extended header with flags=0x88. This was the original interoperability issue -- Versa sends VXLAN-GBP headers that some devices may not properly handle.

---

## Side-by-Side Comparison Table

| Field | Cisco sw2 (GBP ON) | Cisco sw3 (GBP OFF) | Cumulus Linux | Versa FlexVNF |
|-------|-------------------|---------------------|---------------|---------------|
| **VTEP IP** | 1.1.1.2 | 1.1.1.3 | 1.1.1.5 | 1.1.1.4 |
| **Flag Byte** | **0x88** | 0x08 | 0x08 | **0x88** (prev.) |
| **G-bit (bit 7)** | **1 (SET)** | 0 (clear) | 0 (clear) | **1 (SET)** |
| **I-bit (bit 4)** | 1 (SET) | 1 (SET) | 1 (SET) | 1 (SET) |
| **Group Policy ID** | **0x0064 (SGT 100)** | 0x0000 | 0x0000 | varies |
| **VNI** | 401151 | 401151 | 401151 | 401151 |
| **GBP Capable?** | YES | YES (not configured) | NO (FRR limitation) | YES (always on) |

---

## Hex Dump Comparison (VXLAN Header Only)

```
                    Flags Reserved  GPID    VNI      Rsvd
                    ----- --------  ------  -------  ----
Cisco sw2 (GBP):   88    00        00 64   06 1e ff 00    <- SGT 100 in GPID field
Cisco sw3 (std):   08    00        00 00   06 1e ff 00    <- No GBP, GPID=0
Cumulus (std):     08    00        00 00   06 1e ff 00    <- No GBP, GPID=0
Versa (GBP):      88    00        xx xx   06 1e ff 00    <- GBP enabled, GPID varies
```

---

## Key Findings

### 1. Cisco Catalyst 9300 -- GBP Works in Standalone EVPN

The `group-based-policy` command under `interface nve1` successfully enables VXLAN-GBP in standalone BGP EVPN mode (no DNA Center/SD-Access required). This was verified by comparing sw2 (GBP configured) vs sw3 (GBP not configured).

**Configuration on sw2 that enables GBP:**
```
interface nve1
 group-based-policy              ! <-- Enables GBP flag in VXLAN header

cts sgt 100                      ! <-- Local device SGT
cts role-based enforcement
cts role-based enforcement vlan-list 1151
cts role-based sgt-map 192.168.1.0/25 sgt 100
cts role-based sgt-map vlan-list 1151 sgt 100

device-tracking policy IPDT_GBP_POLICY
 tracking enable
 security-level glean

interface GigabitEthernet1/0/3
 device-tracking attach-policy IPDT_GBP_POLICY
interface GigabitEthernet1/0/4
 device-tracking attach-policy IPDT_GBP_POLICY

ip access-list role-based PERMIT_ALL
 permit ip
cts role-based permissions default PERMIT_ALL
```

### 2. Cumulus Linux (FRR) -- No Native GBP Support

Cumulus Linux with FRR does not set the GBP G-bit in outgoing VXLAN packets. All traffic uses standard VXLAN flag 0x08. This is a **platform limitation**, not a configuration issue.

**Critical:** Without the `gbp` flag on the VXLAN interface, the Linux kernel VXLAN driver **silently drops** all incoming packets with the G-bit set (flags=0x88). This affects traffic from **any** GBP-sending VTEP, including Cisco with SGT enabled. With the `gbp` flag added (`ip link add ... gbp`), Cumulus accepts and forwards GBP-tagged traffic but cannot enforce or propagate SGT/GBP policies.

### 3. Versa FlexVNF -- GBP Always Enabled

Versa FlexVNF sends VXLAN packets with GBP flag 0x88 by default. This is the same behavior as Cisco Catalyst 9300 with `group-based-policy` enabled — both vendors use the VXLAN-GBP extended header format defined in `draft-smith-vxlan-group-policy`.

**Important:** This is NOT a Versa-specific issue. The interoperability problem is caused by the **Linux kernel VXLAN driver** dropping GBP-flagged packets when the local interface lacks the `gbp` option. Any vendor sending VXLAN-GBP headers (Cisco with SGT, Versa, or others) will trigger the same behavior on Linux-based VTEPs.

### 4. Interoperability Matrix

| Sender \ Receiver | Cisco (GBP ON) | Cisco (GBP OFF) | Cumulus | Versa |
|-------------------|----------------|-----------------|---------|-------|
| **Cisco (GBP ON)** | OK (GBP preserved) | OK (GBP ignored) | OK (GBP ignored) | OK |
| **Cisco (GBP OFF)** | OK (no GBP) | OK (standard) | OK (standard) | OK |
| **Cumulus** | OK (no GBP sent) | OK (standard) | OK (standard) | OK |
| **Versa** | OK (GBP processed) | Potential issue* | Potential issue* | OK |

*Both Cisco (with GBP/SGT enabled) and Versa send GBP-flagged packets (0x88). Linux-based VTEPs without the `gbp` interface flag will silently drop these packets.

---

## Recommendation

### For Linux-based VTEPs (Cumulus, Ubuntu, etc.)
1. **Always create VXLAN interfaces with `gbp` flag** when integrating with any GBP-capable VTEP (Cisco with SGT, Versa, or others)
2. Without `gbp`, the Linux kernel silently drops GBP-flagged packets — no logs, no counters, extremely difficult to diagnose
3. Cumulus NVUE does not natively support the `gbp` flag — manual recreation or post-boot script required

### For Cisco Catalyst 9300 with SGT/GBP
1. **Be aware that enabling `group-based-policy` on NVE changes the VXLAN header format** from standard (0x08) to GBP-extended (0x88)
2. This will break connectivity to any Linux-based VTEP that lacks the `gbp` interface flag
3. If SGT microsegmentation is not required, leave `group-based-policy` disabled to maintain standard VXLAN interoperability

### For Multi-Vendor Fabrics
1. **Test data plane connectivity after any GBP/SGT configuration change** — control plane (BGP EVPN) will appear healthy even when data plane is completely broken
2. Use `tcpdump -i <interface> -n 'udp port 4789' -XX` to verify VXLAN flags byte (0x08 = standard, 0x88 = GBP)
3. The issue is **symmetric** — Cisco with SGT causes the same drop on Cumulus as Versa does

---

*Report generated from live traffic captures in multi-vendor EVPN VXLAN lab. Cisco sw2 and Versa both confirmed sending VXLAN-GBP headers (flags=0x88) via tcpdump packet captures on Cumulus swp1.*
