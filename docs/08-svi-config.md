# Step 8: SVI (Switched Virtual Interface) Configuration

SVIs provide Layer 3 gateway functionality for VLANs in the VRF.

## Overview

Two types of SVIs:
1. **Core SVI (VLAN 1101)**: Transit interface for L3VNI
2. **Access SVIs (VLAN 1151-1153)**: Default gateways for hosts

## Configuration

### sw2 (Leaf VTEP)

```
! Core SVI for L3VNI (Transit VLAN)
interface Vlan1101
 description SWIforVRF1
 vrf forwarding VRF-1
 ip unnumbered Loopback0
 no autostate
!
! Access SVIs (Host Gateways)
interface Vlan1151
 vrf forwarding VRF-1
 ip address 192.168.1.2 255.255.255.128
!
interface Vlan1152
 vrf forwarding VRF-1
 ip address 192.168.2.2 255.255.255.128
!
interface Vlan1153
 vrf forwarding VRF-1
 ip address 192.168.3.3 255.255.255.128
```

### sw3 (Leaf VTEP)

```
! Core SVI for L3VNI
interface Vlan1101
 description SWIforVRF1
 vrf forwarding VRF-1
 ip unnumbered Loopback0
 no autostate
!
! Access SVIs (Host Gateways)
interface Vlan1151
 vrf forwarding VRF-1
 ip address 192.168.1.3 255.255.255.128
!
interface Vlan1152
 vrf forwarding VRF-1
 ip address 192.168.2.3 255.255.255.128
!
interface Vlan1153
 vrf forwarding VRF-1
 ip address 192.168.3.3 255.255.255.128
```

## Key Points

### Core SVI (VLAN 1101)

1. **ip unnumbered Loopback0**: No IP needed, uses Loopback0 for routing
2. **no autostate**: Keep interface up even without physical ports in VLAN
3. **Purpose**: Provides next-hop for EVPN Type-5 routes

### Access SVIs (VLAN 1151-1153)

1. **Different IP on each VTEP**: sw2 uses .2, sw3 uses .3
2. **VRF forwarding**: Associates SVI with tenant VRF
3. **Subnet**: /25 (255.255.255.128) = 126 usable hosts per VLAN

### Anycast Gateway (Optional Enhancement)

For seamless host mobility, configure same IP on all VTEPs:

```
! On both sw2 and sw3
interface Vlan1151
 vrf forwarding VRF-1
 ip address 192.168.1.1 255.255.255.128
 mac-address 0000.1111.1111
```

**Note**: Anycast gateway requires same IP AND same MAC on all VTEPs.

## Subnet Allocation

| VLAN | Subnet | Gateway | Usable Range |
|------|--------|---------|--------------|
| 1151 | 192.168.1.0/25 | .2/.3 | 192.168.1.1 - 192.168.1.126 |
| 1152 | 192.168.2.0/25 | .2/.3 | 192.168.2.1 - 192.168.2.126 |
| 1153 | 192.168.3.0/25 | .2/.3 | 192.168.3.1 - 192.168.3.126 |

## Verification

```
! Show VRF interfaces
show ip interface brief vrf VRF-1

! Check SVI status
show interface vlan 1151

! View VRF routing table
show ip route vrf VRF-1

! Check ARP table
show ip arp vrf VRF-1
```

### Expected Output

```
sw2# show ip interface brief vrf VRF-1
Interface              IP-Address      OK? Method Status                Protocol
Vlan1101               1.1.1.2         YES TFTP   up                    up
Vlan1151               192.168.1.2     YES TFTP   up                    up
Vlan1152               192.168.2.2     YES TFTP   up                    up
Vlan1153               192.168.3.2     YES TFTP   up                    up
GigabitEthernet1/0/5   192.168.220.2   YES TFTP   up                    up
```

## Next Step

[Step 9: Verification Commands](09-verification.md)
