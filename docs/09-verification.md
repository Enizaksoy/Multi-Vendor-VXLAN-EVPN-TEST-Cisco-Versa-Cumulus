# Step 9: Verification Commands

This guide provides commands to verify the EVPN VXLAN fabric is working correctly.

## 1. Underlay Verification

### OSPF

```
! Check OSPF neighbors
show ip ospf neighbor

! Verify OSPF routes
show ip route ospf

! Test loopback reachability
ping 1.1.1.2 source loopback 0
ping 1.1.1.3 source loopback 0
```

## 2. BGP EVPN Verification

### BGP Neighbors

```
! Check L2VPN EVPN neighbors
show bgp l2vpn evpn summary

! Expected output:
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4  6500     500     510       85    0    0 04:30:15       24
1.1.1.3         4  6500     500     510       85    0    0 04:30:12       24
```

### EVPN Routes

```
! Show all EVPN routes
show bgp l2vpn evpn

! Show Type-2 routes (MAC/IP)
show bgp l2vpn evpn route-type 2

! Show Type-3 routes (Inclusive Multicast)
show bgp l2vpn evpn route-type 3

! Show Type-5 routes (IP Prefix)
show bgp l2vpn evpn route-type 5

! Show routes for specific VNI
show bgp l2vpn evpn vni 401151
```

## 3. VXLAN/NVE Verification

### NVE Interface

```
! Check NVE interface status
show interface nve1

! Expected: up/up with VXLAN encapsulation
```

### NVE Peers

```
! Show discovered VTEP peers
show nve peers

! Expected output:
Interface  VNI      Type Peer-IP          RMAC              eVNI     state
nve1       50101    L3CP 1.1.1.3          aabb.cc00.0300    50101    UP
nve1       401151   L2CP 1.1.1.3          3                 401151   UP
```

### VNI Status

```
! Show VNI details
show nve vni

! Show VNI counters
show nve vni counters
```

## 4. L2VPN EVPN Verification

```
! EVPN summary
show l2vpn evpn summary

! EVPN EVI details
show l2vpn evpn evi detail

! EVPN MAC table
show l2vpn evpn mac

! EVPN MAC for specific EVI
show l2vpn evpn mac evi 1151
```

## 5. MAC Address Table

```
! Show MAC address table
show mac address-table

! Show MAC for specific VLAN
show mac address-table vlan 1151

! Show MAC learning type (local vs EVPN)
show l2vpn evpn mac
```

## 6. VRF Routing Verification

```
! Show VRF routes
show ip route vrf VRF-1

! Show BGP VRF routes
show ip bgp vpnv4 vrf VRF-1

! Show ARP table
show ip arp vrf VRF-1
```

## 7. End-to-End Connectivity Test

### From sw2 (VLAN 1151)

```
! Ping host on sw3 in same VLAN (L2 extension)
ping vrf VRF-1 192.168.1.103

! Ping host in different VLAN (inter-VLAN via VXLAN)
ping vrf VRF-1 192.168.2.103
```

### Traceroute

```
! Trace path to remote host
traceroute vrf VRF-1 192.168.1.103
```

## 8. Troubleshooting Commands

### Debug Commands (Use with caution)

```
! Debug NVE events
debug nve peer

! Debug EVPN
debug l2vpn evpn all

! Debug BGP EVPN updates
debug bgp l2vpn evpn updates
```

### Common Issues

| Symptom | Check | Likely Cause |
|---------|-------|--------------|
| NVE peers not forming | show nve peers | OSPF/BGP not converged |
| MAC not learning | show l2vpn evpn mac | RT mismatch |
| Inter-VLAN routing fails | show nve vni | L3VNI not configured |
| BUM flooding issues | show nve vni | Replication type mismatch |

## 9. Quick Health Check Script

Run these commands in sequence for a quick health check:

```
show ip ospf neighbor
show bgp l2vpn evpn summary
show nve peers
show nve vni
show l2vpn evpn summary
show ip route vrf VRF-1
```

## 10. Useful Packet Capture

```
! Monitor VXLAN traffic on NVE
monitor capture VXLAN interface nve1 both

! Start capture
monitor capture VXLAN start

! View captured packets
show monitor capture VXLAN buffer brief

! Stop capture
monitor capture VXLAN stop
```
