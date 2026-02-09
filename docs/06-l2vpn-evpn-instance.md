# Step 6: L2VPN EVPN Instance Configuration

L2VPN EVPN instances define how VLANs participate in the EVPN control plane.

## Overview

Each L2VNI (access VLAN) needs:
- Global L2VPN EVPN settings
- Per-VLAN EVPN instance with route targets

## Configuration

### Global L2VPN EVPN Settings (sw2 and sw3)

```
l2vpn evpn
 replication-type ingress
 router-id Loopback0
```

### Per-VLAN EVPN Instances (sw2 and sw3)

```
l2vpn evpn instance 1151 vlan-based
 encapsulation vxlan
 route-target export 1151:1
 route-target import 1151:1
!
l2vpn evpn instance 1152 vlan-based
 encapsulation vxlan
 route-target export 1152:1
 route-target import 1152:1
!
l2vpn evpn instance 1153 vlan-based
 encapsulation vxlan
 route-target export 1153:1
 route-target import 1153:1
```

## Key Points

1. **replication-type ingress**: Use ingress replication (head-end replication) for BUM traffic
   - Alternative: `multicast` (requires PIM infrastructure)

2. **router-id Loopback0**: EVPN router ID matches BGP/OSPF router ID

3. **vlan-based**: Each VLAN is a separate broadcast domain (vs. bundle-based)

4. **Route Targets per EVI**:
   - Export RT: What other VTEPs should accept
   - Import RT: What this VTEP accepts from others

### Replication Types Comparison

| Type | Pros | Cons |
|------|------|------|
| Ingress | Simple, no multicast infra needed | More replication at source |
| Multicast | Efficient for large BUM traffic | Requires PIM infrastructure |

### Route Target Design

For each VLAN, use format `VLAN:1`:
- VLAN 1151 -> RT 1151:1
- VLAN 1152 -> RT 1152:1
- VLAN 1153 -> RT 1153:1

This ensures only VTEPs with the same VLAN share MAC/IP information.

## Verification

```
! Show L2VPN EVPN summary
show l2vpn evpn summary

! Show EVPN EVI details
show l2vpn evpn evi detail

! Show EVPN MAC table
show l2vpn evpn mac
```

### Expected Output

```
sw2# show l2vpn evpn evi detail

EVPN instance:       1151 (VLAN Based)
  RD:                1.1.1.2:1151 (auto)
  Import-RTs:        1151:1
  Export-RTs:        1151:1
  Per-EVI Label:     none
  State:             Established
  Encapsulation:     vxlan
  Replication Type:  Ingress (global)

EVPN instance:       1152 (VLAN Based)
  RD:                1.1.1.2:1152 (auto)
  Import-RTs:        1152:1
  Export-RTs:        1152:1
  ...
```

## Next Step

[Step 7: NVE Interface](07-nve-interface.md)
