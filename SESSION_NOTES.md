# EVPN VXLAN Multi-Vendor Lab - Session Notes
**Date:** February 2026
**Environment:** Lab / Proof of Concept

---

## 1. Lab Topology

```
                    +-----------+
                    |  Spine/RR |
                    | (sw1/sw3) |
                    +-----------+
                   /      |      \
         +--------+  +--------+  +---------+
         | sw2    |  | sw3    |  | Cumulus  |
         | Cisco  |  | Cisco  |  | Linux   |
         | 9300   |  | 9300   |  | (NVIDIA)|
         +--------+  +--------+  +---------+
         1.1.1.2     1.1.1.3      1.1.1.5
         |            |            |
     +--------+  +--------+  +---------+
     | Versa  |  |        |  | Client  |
     |FlexVNF |  |        |  | PC      |
     +--------+  +--------+  +---------+
     1.1.1.4                  192.168.1.12
     192.168.1.10
```

### VTEP List
| Device    | Loopback (VTEP) | Mgmt IP         | Role          |
|-----------|----------------|-----------------|---------------|
| sw1       | 1.1.1.1        | 192.168.20.77   | Spine / RR    |
| sw2       | 1.1.1.2        | 192.168.20.76   | Leaf (Cisco)  |
| sw3       | 1.1.1.3        | 192.168.20.81   | Leaf (Cisco)  |
| Versa     | 1.1.1.4        | 192.168.20.166  | Leaf (Versa)  |
| Cumulus   | 1.1.1.5        | 192.168.20.172  | Leaf (Cumulus)|

### Client IPs (VLAN 1151 / VNI 401151)
| Device      | IP Address      | Connected To |
|------------|-----------------|--------------|
| sw2 SVI    | 192.168.1.22/25 | sw2          |
| sw3 SVI    | 192.168.1.23/25 | sw3          |
| Versa vni  | 192.168.1.10/25 | Versa        |
| Cumulus SVI| 192.168.1.15/25 | Cumulus      |
| Client PC  | 192.168.1.12/25 | Cumulus swp2 |
| sw2 client | 192.168.1.102   | sw2          |
| sw3 client | 192.168.1.103   | sw3          |

### Credentials
| Device  | Username | Password     | Notes                           |
|---------|----------|-------------|----------------------------------|
| Cisco   | cisco    | cisco       | disabled_algorithms for SSH       |
| Versa   | admin    | versa123    | Must type `cli` after SSH login  |
| Cumulus  | cumulus  | Versa@123!! | NVUE (nv set) for config         |

---

## 2. Key Technical Details

### VXLAN Configuration
- **VLAN:** 1151
- **VNI:** 401151
- **Replication:** Ingress (Head-End Replication / HER)
- **VXLAN Port:** UDP 4789
- **BGP:** iBGP with Route Reflector (sw1)
- **Underlay:** OSPF area 0

### Vendor-Specific Notes

#### Cisco IOS-XE (sw2/sw3)
- `l2vpn evpn instance 1151 vlan-based` + `member vni 401151`
- `interface nve1` with `member vni 401151 ingress-replication`
- SSH needs: `disabled_algorithms={'pubkeys': ['rsa-sha2-256', 'rsa-sha2-512']}`

#### Versa FlexVNF
- SSH login gives bash shell, must type `cli` to enter CLI mode
- `bridge-options { l2vpn-service-type vlan }`
- Uses DTVI (Dynamic Tunnel Virtual Interface) per remote VTEP
- Sends VXLAN-GBP headers (flags 0x88 instead of standard 0x08)
- Route Target: exports RT 2:2 + RT 1151:1

#### Cumulus Linux (NVIDIA)
- NVUE commands (`nv set ...`)
- Single bridge `br_default` with `bridge-vlan-aware yes`
- Single VXLAN device `vxlan48` with `vnifilter` mode
- `nv set bridge domain br_default vlan 1151 vni 401151`
- FRR (vtysh) for routing verification

---

## 3. Issues Found & Fixed

### Issue 1: VXLAN-GBP Flag Mismatch (CRITICAL)
**Problem:** Any VTEP sending VXLAN-GBP extended headers (flags=0x88) causes
packet drops on Cumulus Linux. This affects BOTH:
- **Cisco Catalyst 9300** with `group-based-policy` + CTS/SGT enabled (sends SGT tag in GPID field, flags=0x88)
- **Versa FlexVNF** (sends GBP headers by default, flags=0x88)

Cumulus vxlan48 created by NVUE does NOT have GBP. Linux kernel drops packets
with unexpected GBP field. This is a **Linux kernel VXLAN driver behavior**,
not a vendor-specific issue.

**Symptom:** 100% data plane packet loss from any GBP-sending VTEP to Cumulus.
Underlay pings work. BGP EVPN sessions established. NVE peers visible.

**Fix:** Manually recreate vxlan48 with GBP:
```bash
sudo ip link del vxlan48
sudo ip link add vxlan48 type vxlan id 401151 local 1.1.1.5 dstport 4789 nolearning ttl 64 gbp
sudo ip link set vxlan48 mtu 9216
sudo ip link set vxlan48 up
sudo ip link set vxlan48 master br_default
sudo /sbin/bridge vlan del vid 1 dev vxlan48
sudo /sbin/bridge vlan add vid 1151 dev vxlan48 pvid untagged
sudo /sbin/bridge link set dev vxlan48 learning off
sudo /sbin/bridge link set dev vxlan48 neigh_suppress on
sudo /sbin/bridge link set dev vxlan48 vlan_tunnel on
```

**WARNING:** This fix is NOT persistent! Lost after reboot or `ifreload -a`.
NVUE does not support GBP natively. Need post-boot script or /etc/network/interfaces edit.

**Script:** `scripts/versa_cumulus_fix.py`

### Issue 2: VLAN-VNI Mapping after VXLAN Recreation
**Problem:** After deleting and recreating vxlan48, default VLAN 1 becomes PVID.
VNI 401151 maps to VLAN 1 instead of VLAN 1151.

**Fix:** `bridge vlan del vid 1 dev vxlan48` then `bridge vlan add vid 1151 dev vxlan48 pvid untagged`

### Issue 3: OSPF MTU Mismatch
**Problem:** Cisco uses 1550, Cumulus uses 9216. OSPF stuck in ExStart.

**Fix:** `ip ospf mtu-ignore` on both sides.

### Issue 4: Cumulus NVUE OSPF Context
**Problem:** OSPF daemon won't activate with standard commands.

**Fix:** `nv set vrf default router ospf enable on`

### Issue 5: Versa Route Target
**Problem:** Versa exports RT 2:2 in addition to RT 1151:1.

**Fix:** All VTEPs import RT 1151:1. Optionally also import RT 2:2.

### Issue 6: swp2 Access vs Trunk Mode (Client Connectivity)
**Problem:** Client PC (192.168.1.12) connected to Cumulus swp2 uses tagged
VLAN 1151 (subinterface). When swp2 set as access port (PVID 1151), Cumulus
sends replies untagged. Client expects tagged frames and drops them.

**Symptom:** Can ping Cumulus SVI but not client behind it. Client can't ping Cumulus.

**Investigation Steps:**
1. Checked `bridge vlan show dev swp2` - PVID was VLAN 1
2. Changed to access port (PVID 1151) - BROKE arping
3. User clarified: "cumulus untagged gonderiyor client tag li gonderiyor"
4. Reverted to trunk mode

**Fix:** Keep swp2 as trunk (default):
- VLAN 1: PVID Egress Untagged (native)
- VLAN 1151: tagged (no PVID, no untagged flag)

**Scripts:**
- `scripts/cumulus_swp2_fix.py` - Initial diagnosis
- `scripts/cumulus_swp2_access.py` - Access port attempt (failed)
- `scripts/cumulus_swp2_trunk.py` - Final fix (trunk mode)

---

## 4. Cross-Vendor Concept Mapping

### VLAN-Based EVPN Service Definition
| Function           | Cisco IOS-XE                              | Versa FlexVNF                              | Cumulus Linux                           |
|-------------------|-------------------------------------------|--------------------------------------------|-----------------------------------------|
| EVPN Service Type | `l2vpn evpn instance 1151 vlan-based`     | `bridge-options { l2vpn-service-type vlan }`| Implicit (bridge-vlan-aware + VNI map)  |
| VLAN-Aware Bridge | Native VLAN database                      | Bridge-domain per VLAN                     | `nv set bridge domain br_default`       |
| VNI Binding       | `member vni 401151` under VLAN config     | `encapsulation vxlan { vni 401151 }`       | `vlan 1151 vni 401151`                  |
| VXLAN Interface   | `interface nve1` + `member vni`           | One DTVI per remote VTEP                   | Single `vxlan48` with vnifilter         |
| Replication       | `replication-type ingress`                | Ingress replication (auto)                 | HER via FRR/EVPN                        |
| Bridge Model      | Per-VLAN EVPN instance                    | Per bridge-domain                          | Single bridge, all VLANs                |

### Key Difference
Cisco and Versa explicitly declare "VLAN-based L2VPN service". Cumulus does it
implicitly - bridge-vlan-aware + VLAN-VNI mapping automatically generates
EVPN Type-2 (MAC/IP) and Type-3 (IMET) routes.

---

## 5. Word Document (convert_to_docx.py)

### Location
`docs/convert_to_docx.py` -> generates `docs/Versa-Cisco-EVPN-VXLAN-Integration.docx`

### Features Added
1. **Syntax Highlighting** - Terminal-style colored output:
   - IP addresses: teal blue
   - MAC addresses: amber
   - Status UP/DOWN: green/red
   - Prompts (sw1#, cumulus$): green bold
   - Interface names: orange-brown
   - Config keywords: blue
   - VNI numbers: purple
   - Comments: gray

2. **Landscape Orientation** - Section 14 (comparison tables) in landscape mode

3. **Confluence Bracket Fix** - Zero-width space (U+200B) after `[` to prevent
   Confluence wiki macro interpretation

4. **Save Fallback** - Tries suffixes ['', '_v2', '_v3', '_v4'] if file locked

### Running
```bash
cd C:\Claude\EVPN-VXLAN-Catalyst9K\docs
python convert_to_docx.py
```

---

## 6. GBP Persistence - TODO for Next Session

The manual GBP fix is NOT persistent. Options to make it survive reboot:

### Option A: Edit /etc/network/interfaces
Add a `post-up` hook to the vxlan48 stanza that recreates with GBP.

### Option B: Systemd Service
Create a systemd unit that runs after networking to recreate vxlan48 with GBP.

### Option C: Custom ifupdown2 script
Put a script in `/etc/network/if-up.d/` that checks and fixes GBP.

**None of these have been implemented yet.**

---

## 7. Verification Commands Cheat Sheet

### Cumulus
```bash
# Check GBP on vxlan48
ip -d link show vxlan48 | grep gbp

# Check VLAN-VNI mapping
sudo /sbin/bridge vlan show dev vxlan48

# Check EVPN VNI
sudo vtysh -c "show evpn vni 401151"

# Check swp2 VLAN mode
sudo /sbin/bridge vlan show dev swp2

# Check BGP EVPN
sudo vtysh -c "show bgp l2vpn evpn"
```

### Cisco
```
show nve peers
show l2vpn evpn evi 1151 detail
show bgp l2vpn evpn summary
show vxlan tunnel
```

### Versa (after `cli` command)
```
show bridge mac-table brief routing-instance VERSA-default-switch | nomore
show bridge ingress-table routing-instance VERSA-default-switch | nomore
show bgp summary
ping 1.1.1.5 routing-instance Underlay count 3
```

---

## 8. Important Scripts

| Script | Purpose | Status |
|--------|---------|--------|
| `versa_cumulus_fix.py` | Check GBP, fix vxlan48, test all pings | Working - run after reboot |
| `cumulus_swp2_trunk.py` | Revert swp2 to trunk mode | Working |
| `cumulus_swp2_access.py` | Set swp2 as access port (DON'T USE) | Broke client |
| `convert_to_docx.py` | Generate Word document | Working |

---

## 9. Final State (Last Verified)

All connectivity working:
```
192.168.1.22  (sw2 SVI)      -> OK (0% loss)
192.168.1.23  (sw3 SVI)      -> OK (0% loss)
192.168.1.10  (Versa client) -> OK (0% loss)
192.168.1.102 (sw2 client)   -> OK (0% loss)
192.168.1.103 (sw3 client)   -> OK (0% loss)
192.168.1.12  (Cumulus client)-> OK (0% loss)
```

**After reboot:** Run `versa_cumulus_fix.py` to re-enable GBP on vxlan48.
