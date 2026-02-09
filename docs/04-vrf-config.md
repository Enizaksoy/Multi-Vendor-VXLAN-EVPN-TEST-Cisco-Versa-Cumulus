# Step 4: VRF Configuration

VRF (Virtual Routing and Forwarding) provides Layer 3 segmentation for tenant traffic.

## Overview

VRF configuration includes:
- VRF definition with Route Distinguisher (RD)
- Route Targets (RT) for import/export
- Stitching route targets for EVPN-to-VRF route leaking

## Configuration

### sw2 and sw3 (Leaf VTEPs only)

```
vrf definition VRF-1
 rd 10:1101
 !
 address-family ipv4
  route-target export 10:1101
  route-target import 10:1101
  route-target export 10:1101 stitching
  route-target import 10:1101 stitching
 exit-address-family
```

## Key Points

1. **Route Distinguisher (RD)**: Makes routes unique in BGP (format: ASN:number or IP:number)
2. **Route Target (RT)**: Controls route import/export between VRFs
3. **Stitching RT**: Required for EVPN Type-5 (IP prefix) routes to be imported into VRF

### RD vs RT

| Attribute | Purpose | Scope |
|-----------|---------|-------|
| RD | Makes routes unique | Per-device |
| RT | Controls route distribution | Network-wide |

## VRF Route Target Design

For this fabric:
- **RD**: 10:1101 (same across all leafs for simplicity)
- **RT**: 10:1101 (all leafs import/export same RT for full mesh connectivity)

### Multi-Tenant Design (Example)

```
! Tenant A
vrf definition TENANT-A
 rd 100:1
 address-family ipv4
  route-target export 100:1
  route-target import 100:1
  route-target export 100:1 stitching
  route-target import 100:1 stitching

! Tenant B
vrf definition TENANT-B
 rd 200:1
 address-family ipv4
  route-target export 200:1
  route-target import 200:1
  route-target export 200:1 stitching
  route-target import 200:1 stitching
```

## Verification

```
! Show VRF details
show vrf detail VRF-1

! Check VRF routing table
show ip route vrf VRF-1

! View VRF interfaces
show ip interface brief vrf VRF-1
```

### Expected Output

```
sw2# show vrf detail VRF-1

VRF VRF-1 (VRF Id = 1); default RD 10:1101; default VPNID <not set>
  Interfaces:
    Gi1/0/5                  Vl1101                   Vl1151
    Vl1152                   Vl1153
  Address family ipv4 unicast (Table ID = 0x1):
    Export VPN route-target communities
      RT:10:1101                 RT:10:1101
    Import VPN route-target communities
      RT:10:1101                 RT:10:1101
```

## Next Step

[Step 5: VLAN and VNI Mapping](05-vlan-vni-mapping.md)
