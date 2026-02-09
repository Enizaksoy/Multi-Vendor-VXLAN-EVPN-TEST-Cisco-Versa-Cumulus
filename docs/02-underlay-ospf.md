# Step 2: Underlay Network (OSPF)

The underlay network provides IP reachability between VTEP loopbacks using OSPF.

## Overview

OSPF is used to:
- Advertise loopback addresses between all switches
- Provide optimal path selection for VXLAN traffic
- Enable fast convergence with tuned timers

## Physical Connectivity

```
sw1 Gi1/0/1 (12.12.12.1) <---> (12.12.12.2) Gi1/0/1 sw2
sw1 Gi1/0/2 (12.12.12.5) <---> (12.12.12.6) Gi1/0/1 sw3
```

## Configuration

### sw1 (Spine)

```
interface GigabitEthernet1/0/1
 no switchport
 ip address 12.12.12.1 255.255.255.252
 ip pim sparse-mode
 ip ospf network point-to-point
 ip ospf 100 area 100
!
interface GigabitEthernet1/0/2
 no switchport
 ip address 12.12.12.5 255.255.255.252
 ip pim sparse-mode
 ip ospf network point-to-point
 ip ospf 100 area 100
!
router ospf 100
 router-id 1.1.1.1
 auto-cost reference-bandwidth 100000
 timers throttle spf 10 100 5000
 timers lsa arrival 80
```

### sw2 (Leaf)

```
interface GigabitEthernet1/0/1
 no switchport
 ip address 12.12.12.2 255.255.255.252
 ip pim sparse-mode
 ip ospf network point-to-point
 ip ospf 100 area 100
!
router ospf 100
 router-id 1.1.1.2
 auto-cost reference-bandwidth 100000
 timers throttle spf 10 100 5000
 timers lsa arrival 80
```

### sw3 (Leaf)

```
interface GigabitEthernet1/0/1
 no switchport
 ip address 12.12.12.6 255.255.255.252
 ip pim sparse-mode
 ip ospf network point-to-point
 ip ospf 100 area 100
!
router ospf 100
 router-id 1.1.1.3
 auto-cost reference-bandwidth 100000
 timers throttle spf 10 100 5000
 timers lsa arrival 80
```

## Key Points

1. **Point-to-point network type**: Faster convergence, no DR/BDR election
2. **Reference bandwidth 100000**: Ensures proper cost calculation for high-speed links
3. **SPF throttle timers**: Fast initial convergence (10ms) with backoff
4. **Area 0**: Loopbacks in backbone area
5. **Area 100**: Inter-switch links (could also be area 0)

## Verification

```
! Check OSPF neighbors
show ip ospf neighbor

! Verify loopback reachability
ping 1.1.1.2 source loopback 0
ping 1.1.1.3 source loopback 0

! View OSPF routes
show ip route ospf

! Check OSPF database
show ip ospf database
```

### Expected Output

```
sw1# show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.2           0   FULL/  -        00:00:35    12.12.12.2      GigabitEthernet1/0/1
1.1.1.3           0   FULL/  -        00:00:39    12.12.12.6      GigabitEthernet1/0/2
```

## Next Step

[Step 3: BGP EVPN Overlay](03-bgp-evpn-overlay.md)
