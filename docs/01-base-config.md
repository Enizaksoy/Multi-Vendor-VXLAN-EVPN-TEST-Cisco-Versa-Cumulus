# Step 1: Base Configuration

This step covers the basic device configuration including hostname, IP routing, and loopback interfaces.

## Overview

Each device needs:
- Unique hostname
- IP routing enabled
- Loopback interface for VTEP/BGP router-ID
- System MTU set for VXLAN overhead (9000+ recommended)

## Configuration

### sw1 (Spine)

```
hostname sw1
!
ip routing
!
system mtu 8978
!
interface Loopback0
 description OSPF+BGP+NVE
 ip address 1.1.1.1 255.255.255.255
 ip pim sparse-mode
 ip ospf 100 area 0
```

### sw2 (Leaf)

```
hostname sw2
!
ip routing
!
system mtu 8978
!
interface Loopback0
 description OSPF+BGP+NVE
 ip address 1.1.1.2 255.255.255.255
 ip pim sparse-mode
 ip ospf 100 area 0
```

### sw3 (Leaf)

```
hostname sw3
!
ip routing
!
system mtu 8978
!
interface Loopback0
 description OSPF+BGP+NVE
 ip address 1.1.1.3 255.255.255.255
 ip pim sparse-mode
 ip ospf 100 area 0
```

## Key Points

1. **System MTU**: Set to at least 8978 to accommodate VXLAN 50-byte overhead
2. **Loopback0**: Used as:
   - OSPF router-ID
   - BGP router-ID
   - NVE source interface (VTEP IP)
3. **PIM sparse-mode**: Required if using multicast for BUM traffic replication

## Verification

```
show ip interface brief
show running-config | section Loopback
```

## Next Step

[Step 2: Underlay (OSPF)](02-underlay-ospf.md)
