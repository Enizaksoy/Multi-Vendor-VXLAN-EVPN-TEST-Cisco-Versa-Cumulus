# Step 7: NVE Interface Configuration

The NVE (Network Virtualization Edge) interface is the VXLAN Tunnel Endpoint (VTEP).

## Overview

NVE interface configuration includes:
- Source interface (Loopback for VTEP IP)
- Host reachability protocol (BGP for EVPN)
- VNI membership (L2VNI and L3VNI)

## Configuration

### sw2 (Leaf VTEP)

```
interface nve1
 description nve_interface_for_vxlan
 no ip address
 source-interface Loopback0
 host-reachability protocol bgp
 member vni 50101 vrf VRF-1
 member vni 401151 ingress-replication
 member vni 401152 ingress-replication local-routing
 member vni 401153 ingress-replication local-routing
```

### sw3 (Leaf VTEP)

```
interface nve1
 description nve_interface_for_vxlan
 no ip address
 source-interface Loopback0
 host-reachability protocol bgp
 member vni 50101 vrf VRF-1
 member vni 401151 ingress-replication local-routing
 member vni 401152 ingress-replication local-routing
 member vni 401153 ingress-replication local-routing
```

## Key Points

1. **source-interface Loopback0**: VTEP IP address (must be reachable via underlay)

2. **host-reachability protocol bgp**: Use BGP EVPN for MAC/IP learning

3. **member vni 50101 vrf VRF-1**: L3VNI association with VRF for inter-VLAN routing

4. **ingress-replication**: BUM traffic replicated at ingress VTEP
   - Alternative: `mcast-group <IP>` for multicast replication

5. **local-routing**: Enable distributed anycast gateway (routing on local VTEP)
   - Required for hosts to route through local SVI
   - Improves performance by avoiding hairpin routing

### With vs Without local-routing

| Setting | Traffic Flow | Use Case |
|---------|--------------|----------|
| Without local-routing | All routing via spine | Centralized routing |
| With local-routing | Route at local VTEP | Distributed anycast gateway |

## NVE VNI Types

```
! L3VNI - for inter-VLAN routing
member vni 50101 vrf VRF-1

! L2VNI - for Layer 2 extension
member vni 401151 ingress-replication local-routing
```

## Verification

```
! Check NVE interface status
show interface nve1

! Show NVE peers
show nve peers

! Show VNI details
show nve vni

! Show NVE internal info
show nve interface nve1 detail
```

### Expected Output

```
sw2# show nve peers

Interface  VNI      Type Peer-IP          RMAC/Num_RTs   eVNI     state flags UP time
nve1       50101    L3CP 1.1.1.3          aabb.cc00.0300 50101      UP  A/M/4 00:45:22
nve1       401151   L2CP 1.1.1.3          3              401151     UP   N/A  00:45:20
nve1       401152   L2CP 1.1.1.3          3              401152     UP   N/A  00:45:20
nve1       401153   L2CP 1.1.1.3          3              401153     UP   N/A  00:45:20

sw2# show nve vni

Interface  VNI        Multicast-group   VNI state  Mode  VLAN  cfg  vrf
nve1       50101      N/A               Up         L3CP  1101  CLI  VRF-1
nve1       401151     UnicastBGP        Up         L2CP  1151  CLI  N/A
nve1       401152     UnicastBGP        Up         L2CP  1152  CLI  N/A
nve1       401153     UnicastBGP        Up         L2CP  1153  CLI  N/A
```

## Next Step

[Step 8: SVI Configuration](08-svi-config.md)
