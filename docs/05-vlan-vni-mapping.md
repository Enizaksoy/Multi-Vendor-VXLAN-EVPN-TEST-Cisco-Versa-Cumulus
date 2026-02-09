# Step 5: VLAN and VNI Mapping

This step creates VLANs and maps them to VXLAN Network Identifiers (VNIs).

## Overview

Two types of VNIs are used:
1. **L2VNI**: For Layer 2 extension (MAC learning across sites)
2. **L3VNI**: For Layer 3 routing between VLANs (symmetric IRB)

## VLAN/VNI Mapping Table

| VLAN | Name | VNI | Type | Purpose |
|------|------|-----|------|---------|
| 1101 | VRF1_CORE_VLAN | 50101 | L3VNI | Inter-VLAN routing |
| 1151 | VRF1_ACCESS_VLAN | 401151 | L2VNI | Access VLAN 1 |
| 1152 | VRF1_ACCESS_VLAN_2 | 401152 | L2VNI | Access VLAN 2 |
| 1153 | VRF_ACCESS_VLAN_3 | 401153 | L2VNI | Access VLAN 3 |

## Configuration

### sw2 and sw3 (Leaf VTEPs)

```
! Create VLANs
vlan 1101
 name VRF1_CORE_VLAN
!
vlan 1151
 name VRF1_ACCESS_VLAN
!
vlan 1152
 name VRF1_ACCESS_VLAN_2
!
vlan 1153
 name VRF_ACCESS_VLAN_3
!
! Map VLANs to VNIs
vlan configuration 1101
 member vni 50101
!
vlan configuration 1151
 member evpn-instance 1151 vni 401151
!
vlan configuration 1152
 member evpn-instance 1152 vni 401152
!
vlan configuration 1153
 member evpn-instance 1153 vni 401153
```

## Key Points

1. **L3VNI (1101/50101)**:
   - Uses `member vni` (not evpn-instance)
   - Acts as transit VLAN for inter-VLAN routing
   - Required for symmetric IRB

2. **L2VNI (1151-1153)**:
   - Uses `member evpn-instance`
   - Each VLAN gets its own EVPN instance
   - VNI assignment follows pattern: 400000 + VLAN ID

### VNI Numbering Convention

| Type | Range | Example |
|------|-------|---------|
| L3VNI | 50000-59999 | 50101 |
| L2VNI | 400000+ | 401151 |

## Access Port Configuration

```
! Configure access port for VLAN 1151
interface GigabitEthernet1/0/3
 switchport access vlan 1151
 switchport mode access
 spanning-tree portfast
!
! Configure trunk port (all VLANs)
interface GigabitEthernet1/0/4
 switchport mode trunk
```

## Verification

```
! Show VLANs
show vlan brief

! Show VLAN to VNI mapping
show vlan configuration

! Verify VNI assignment
show nve vni
```

### Expected Output

```
sw2# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
1101 VRF1_CORE_VLAN                   active
1151 VRF1_ACCESS_VLAN                 active    Gi1/0/3, Gi1/0/4
1152 VRF1_ACCESS_VLAN_2               active    Gi1/0/4
1153 VRF_ACCESS_VLAN_3                active    Gi1/0/4
```

## Next Step

[Step 6: L2VPN EVPN Instance](06-l2vpn-evpn-instance.md)
