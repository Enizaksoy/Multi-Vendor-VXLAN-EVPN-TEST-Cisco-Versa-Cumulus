# Step 3: BGP EVPN Overlay

BGP with the L2VPN EVPN address family is used to exchange MAC/IP information between VTEPs.

## Overview

This configuration uses:
- iBGP (AS 6500) between all switches
- Full mesh peering (or Route Reflector for scale)
- L2VPN EVPN address family for MAC/IP advertisement
- Peer templates for simplified configuration

## Configuration

### sw1 (Spine - can act as Route Reflector)

```
router bgp 6500
 !
 ! Peer templates for reusability
 template peer-policy spine-peer-policy
  soft-reconfiguration inbound
  send-community both
 exit-peer-policy
 !
 template peer-session spine-peer-session
  remote-as 6500
  description spine-evpn-peer
  log-neighbor-changes
  update-source Loopback0
 exit-peer-session
 !
 bgp router-id interface Loopback0
 bgp log-neighbor-changes
 !
 ! iBGP peers (Leaf switches)
 neighbor 1.1.1.2 inherit peer-session spine-peer-session
 neighbor 1.1.1.3 inherit peer-session spine-peer-session
 !
 address-family l2vpn evpn
  neighbor 1.1.1.2 activate
  neighbor 1.1.1.2 send-community both
  neighbor 1.1.1.2 inherit peer-policy spine-peer-policy
  neighbor 1.1.1.3 activate
  neighbor 1.1.1.3 send-community both
  neighbor 1.1.1.3 inherit peer-policy spine-peer-policy
 exit-address-family
```

### sw2 (Leaf)

```
router bgp 6500
 !
 template peer-policy spine-peer-policy
  soft-reconfiguration inbound
  send-community both
 exit-peer-policy
 !
 template peer-session spine-peer-session
  remote-as 6500
  description spine-evpn-peer
  log-neighbor-changes
  update-source Loopback0
 exit-peer-session
 !
 bgp router-id interface Loopback0
 bgp log-neighbor-changes
 !
 neighbor 1.1.1.1 inherit peer-session spine-peer-session
 neighbor 1.1.1.3 inherit peer-session spine-peer-session
 !
 address-family l2vpn evpn
  neighbor 1.1.1.1 activate
  neighbor 1.1.1.1 send-community both
  neighbor 1.1.1.1 inherit peer-policy spine-peer-policy
  neighbor 1.1.1.3 activate
  neighbor 1.1.1.3 send-community both
  neighbor 1.1.1.3 inherit peer-policy spine-peer-policy
 exit-address-family
```

### sw3 (Leaf)

```
router bgp 6500
 !
 template peer-policy spine-peer-policy
  soft-reconfiguration inbound
  send-community both
 exit-peer-policy
 !
 template peer-session spine-peer-session
  remote-as 6500
  description spine-evpn-peer
  log-neighbor-changes
  update-source Loopback0
 exit-peer-session
 !
 bgp router-id interface Loopback0
 bgp log-neighbor-changes
 !
 neighbor 1.1.1.1 inherit peer-session spine-peer-session
 neighbor 1.1.1.2 inherit peer-session spine-peer-session
 !
 address-family l2vpn evpn
  neighbor 1.1.1.1 activate
  neighbor 1.1.1.1 send-community both
  neighbor 1.1.1.1 inherit peer-policy spine-peer-policy
  neighbor 1.1.1.2 activate
  neighbor 1.1.1.2 send-community both
  neighbor 1.1.1.2 inherit peer-policy spine-peer-policy
 exit-address-family
```

## Key Points

1. **update-source Loopback0**: Use loopback for BGP peering (stable, routable)
2. **send-community both**: Required to send extended communities (Route Targets)
3. **soft-reconfiguration inbound**: Allows policy changes without session reset
4. **Peer templates**: Simplify configuration and ensure consistency

## EVPN Route Types

BGP EVPN uses different route types:

| Type | Name | Purpose |
|------|------|---------|
| 1 | Ethernet Auto-Discovery | Multihoming, fast convergence |
| 2 | MAC/IP Advertisement | MAC and IP address learning |
| 3 | Inclusive Multicast | BUM traffic replication |
| 4 | Ethernet Segment | Multihoming |
| 5 | IP Prefix | Inter-subnet routing |

## Verification

```
! Check BGP neighbors
show bgp l2vpn evpn summary

! View EVPN routes
show bgp l2vpn evpn

! Check specific route types
show bgp l2vpn evpn route-type 2
show bgp l2vpn evpn route-type 3
```

### Expected Output

```
sw2# show bgp l2vpn evpn summary
BGP router identifier 1.1.1.2, local AS number 6500
BGP table version is 45, main routing table version 45

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4         6500     234     245       45    0    0 02:15:32       12
1.1.1.3         4         6500     234     245       45    0    0 02:15:30       12
```

## Next Step

[Step 4: VRF Configuration](04-vrf-config.md)
