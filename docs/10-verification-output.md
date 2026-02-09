# Live Verification Output

This document shows actual output from the running EVPN VXLAN fabric.

## sw2

### `show nve peers`

```
show nve peers
'M' - MAC entry download flag  'A' - Adjacency download flag
'4' - IPv4 flag  '6' - IPv6 flag
Interface  VNI      Type Peer-IP          RMAC/Num_RTs   eVNI     state flags UP time
nve1       50101    L3CP 1.1.1.3          5254.0018.449f 50101      UP  A/M/4 01:23:03
nve1       401151   L2CP 1.1.1.3          3              401151     UP   N/A  01:23:03
nve1       401152   L2CP 1.1.1.3          3              401152     UP   N/A  01:23:03
nve1       401153   L2CP 1.1.1.3          1              401153     UP   N/A  01:00:56
```

### `show nve vni`

```
show nve vni
Interface  VNI        Multicast-group  VNI state  Mode  VLAN  cfg vrf                      
nve1       401153     N/A              Up         L2CP  1153  CLI VRF-1                    
nve1       401152     N/A              Up         L2CP  1152  CLI VRF-1                    
nve1       401151     N/A              Up         L2CP  1151  CLI VRF-1                    
nve1       50101      N/A              Up         L3CP  1101  CLI VRF-1                    
```

### `show bgp l2vpn evpn summary`

```
show bgp l2vpn evpn summary
BGP router identifier 1.1.1.2, local AS number 6500
BGP table version is 62, main routing table version 62
33 network entries using 12936 bytes of memory
42 path entries using 9744 bytes of memory
20/19 BGP path/bestpath attribute entries using 5920 bytes of memory
1 BGP AS-PATH entries using 40 bytes of memory
12 BGP extended community entries using 564 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 29204 total bytes of memory
BGP activity 53/9 prefixes, 75/13 paths, scan interval 60 secs
34 networks peaked at 16:08:40 Feb 9 2026 UTC (00:26:18.205 ago)
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4         6500      99     114       62    0    0 01:24:19        0
1.1.1.3         4         6500     129     125       62    0    0 01:24:03       15
```

### `show bgp l2vpn evpn`

```
show bgp l2vpn evpn
BGP table version is 62, local router ID is 1.1.1.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
              t secondary path, L long-lived-stale,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found
     Network          Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1.1.1.2:1151
 *>   [2][1.1.1.2:1151][0][48][525400C4B4A1][0][*]/20
                      0.0.0.0                            32768 ?
 *>   [2][1.1.1.2:1151][0][48][AABBCC002400][0][*]/20
                      0.0.0.0                            32768 ?
 *>   [2][1.1.1.2:1151][0][48][AABBCC002400][32][192.168.1.22]/24
                      0.0.0.0                            32768 ?
 *>i  [2][1.1.1.2:1151][0][48][AABBCC002700][0][*]/20
                      1.1.1.3                  0    100      0 ?
 *>i  [2][1.1.1.2:1151][0][48][AABBCC002700][32][192.168.1.23]/24
                      1.1.1.3                  0    100      0 ?
Route Distinguisher: 1.1.1.2:1152
 *>   [2][1.1.1.2:1152][0][48][AABBCC002400][0][*]/20
                      0.0.0.0                            32768 ?
 *>   [2][1.1.1.2:1152][0][48][AABBCC002400][32][192.168.2.22]/24
                      0.0.0.0                            32768 ?
 *>i  [2][1.1.1.2:1152][0][48][AABBCC002700][0][*]/20
                      1.1.1.3                  0    100      0 ?
 *>i  [2][1.1.1.2:1152][0][48][AABBCC002700][32][192.168.2.23]/24
                      1.1.1.3                  0    100      0 ?
Route Distinguisher: 1.1.1.2:1153
 *>   [2][1.1.1.2:1153][0][48][AABBCC002400][0][*]/20
                      0.0.0.0                            32768 ?
 *>   [2][1.1.1.2:1153][0][48][AABBCC002400][32][192.168.3.22]/24
                      0.0.0.0                            32768 ?
Route Distinguisher: 1.1.1.3:1151
 *>i  [2][1.1.1.3:1151][0][48][AABBCC002700][0][*]/20
                      1.1.1.3                  0    100      0 ?
 *>i  [2][1.1.1.3:1151][0][48][AABBCC002700][32][192.168.1.23]/24
                      1.1.1.3                  0    100      0 ?
Route Distinguisher: 1.1.1.3:1152
 *>i  [2][1.1.1.3:1152][0][48][AABBCC002700][0][*]/20
                      1.1.1.3                  0    100      0 ?
 *>i  [2][1.1.1.3:1152][0][48][AABBCC002700][32][192.168.2.23]/24
                      1.1.1.3                  0    100      0 ?
Route Distinguisher: 1.1.1.2:1151
 *>   [3][1.1.1.2:1151][0][32][1.1.1.2]/17
                      0.0.0.0                            32768 ?
 *>i  [3][1.1.1.2:1151][0][32][1.1.1.3]/17
                      1.1.1.3                  0    100      0 ?
Route Distinguisher: 1.1.1.2:1152
 *>   [3][1.1.1.2:1152][0][32][1.1.1.2]/17
                      0.0.0.0                            32768 ?
 *>i  [3][1.1.1.2:1152][0][32][1.1.1.3]/17
                      1.1.1.3                  0    100      0 ?
Route Distinguisher: 1.1.1.2:1153
 *>   [3][1.1.1.2:1153][0][32][1.1.1.2]/17
                      0.0.0.0                            32768 ?
 *>i  [3][1.1.1.2:1153][0][32][1.1.1.3]/17
                      1.1.1.3                  0    100      0 ?
Route Distinguisher: 1.1.1.3:1151
 *>i  [3][1.1.1.3:1151][0][32][1.1.1.3]/17
                      1.1.1.3                  0    100      0 ?
Route Distinguisher: 1.1.1.3:1152
 *>i  [3][1.1.1.3:1152][0][32][1.1.1.3]/17
                      1.1.1.3                  0    100      0 ?
Route Distinguisher: 1.1.1.3:1153
 *>i  [3][1.1.1.3:1153][0][32][1.1.1.3]/17
                      1.1.1.3                  0    100      0 ?
Route Distinguisher: 10:1101 (default for vrf VRF-1)
 * i  [5][10:1101][0][24][192.168.20.0]/17
                      1.1.1.3                  0    100      0 100 6505 ?
 *>                    192.168.220.1            0             0 100 6505 ?
 *    [5][10:1101][0][24][192.168.220.0]/17
                      192.168.220.1            0             0 100 6505 ?
 *>                    0.0.0.0                  0         32768 ?
 *>i  [5][10:1101][0][24][192.168.221.0]/17
                      1.1.1.3                  0    100      0 ?
 *                     192.168.220.1            0             0 100 6505 ?
 * i  [5][10:1101][0][25][192.168.1.0]/17
                      1.1.1.3                  0    100      0 ?
 *>                    0.0.0.0                  0         32768 ?
 * i  [5][10:1101][0][25][192.168.2.0]/17
                      1.1.1.3                  0    100      0 ?
 *>                    0.0.0.0                  0         32768 ?
 * i  [5][10:1101][0][25][192.168.3.0]/17
                      1.1.1.3                  0    100      0 ?
 *>                    0.0.0.0                  0         32768 ?
 * i  [5][10:1101][0][32][10.1.1.1]/17
                      1.1.1.3                  0    100      0 100 6505 ?
 *>                    192.168.220.1            0             0 100 6505 ?
 * i  [5][10:1101][0][32][10.1.1.2]/17
                      1.1.1.3                  0    100      0 100 6505 ?
 *>                    192.168.220.1            0             0 100 6505 ?
 * i  [5][10:1101][0][32][10.1.1.3]/17
                      1.1.1.3                  0    100      0 100 6505 ?
 *>                    192.168.220.1            0             0 100 6505 ?
```

### `show l2vpn evpn evi detail`

```
show l2vpn evpn evi detail
EVPN instance:          1151 (VLAN Based)
  RD:                   1.1.1.2:1151 (auto)
  Import-RTs:           1151:1 6500:1151 
  Export-RTs:           1151:1 6500:1151 
  Per-EVI Label:        none
  State:                Established
  Replication Type:     Ingress (global)
  Encapsulation:        vxlan
  IP Local Learn:       Enabled (global)
  Adv. Def. Gateway:    Disabled (global)
  Re-originate RT5:     Disabled
  Adv. Multicast:       Disabled (global)
  AR Flood Suppress:    Enabled (global)
  Vlan:                 1151
    Protected:          False
    Ethernet-Tag:       0
    State:              Established
    Flood Suppress:     Attached
    Core If:            Vlan1101
    Access If:          Vlan1151
    NVE If:             nve1
    RMAC:               5254.0069.ac31
    Core Vlan:          1101
    L2 VNI:             401151
    L3 VNI:             50101
    VTEP IP:            1.1.1.2
    Originating Router: 1.1.1.2
    VRF:                VRF-1
    IPv4 IRB:           Enabled
    IPv6 IRB:           Disabled
    Pseudoports:
      GigabitEthernet1/0/3 service instance 1151
        Routes: 1 MAC, 0 MAC/IP
      GigabitEthernet1/0/4 service instance 1151
        Routes: 1 MAC, 1 MAC/IP
    Peers:
      1.1.1.3
        Routes: 1 MAC, 1 MAC/IP, 1 IMET, 0 EAD
EVPN instance:          1152 (VLAN Based)
  RD:                   1.1.1.2:1152 (auto)
  Import-RTs:           1152:1 6500:1152 
  Export-RTs:           1152:1 6500:1152 
  Per-EVI Label:        none
  State:                Established
  Replication Type:     Ingress (global)
  Encapsulation:        vxlan
  IP Local Learn:       Enabled (global)
  Adv. Def. Gateway:    Disabled (global)
  Re-originate RT5:     Disabled
  Adv. Multicast:       Disabled (global)
  AR Flood Suppress:    Enabled (global)
  Vlan:                 1152
    Protected:          False
    Ethernet-Tag:       0
    State:              Established
    Flood Suppress:     Attached
    Core If:            Vlan1101
    Access If:          Vlan1152
    NVE If:             nve1
    RMAC:               5254.0069.ac31
    Core Vlan:          1101
    L2 VNI:             401152
    L3 VNI:             50101
    VTEP IP:            1.1.1.2
    Originating Router: 1.1.1.2
    VRF:                VRF-1
    IPv4 IRB:           Enabled (Local routing)
    IPv6 IRB:           Disabled
    Pseudoports:
      GigabitEthernet1/0/4 service instance 1152
        Routes: 1 MAC, 1 MAC/IP
    Peers:
      1.1.1.3
        Routes: 1 MAC, 1 MAC/IP, 1 IMET, 0 EAD
EVPN instance:          1153 (VLAN Based)
  RD:                   1.1.1.2:1153 (auto)
  Import-RTs:           6500:1153 1153:1 
  Export-RTs:           6500:1153 1153:1 
  Per-EVI Label:        none
  State:                Established
  Replication Type:     Ingress (global)
  Encapsulation:        vxlan
  IP Local Learn:       Enabled (global)
  Adv. Def. Gateway:    Disabled (global)
  Re-originate RT5:     Disabled
  Adv. Multicast:       Disabled (global)
  AR Flood Suppress:    Enabled (global)
  Vlan:                 1153
    Protected:          False
    Ethernet-Tag:       0
    State:              Established
    Flood Suppress:     Attached
    Core If:            Vlan1101
    Access If:          Vlan1153
    NVE If:             nve1
    RMAC:               5254.0069.ac31
    Core Vlan:          1101
    L2 VNI:             401153
    L3 VNI:             50101
    VTEP IP:            1.1.1.2
    Originating Router: 1.1.1.2
    VRF:                VRF-1
    IPv4 IRB:           Enabled (Local routing)
    IPv6 IRB:           Disabled
    Pseudoports:
      GigabitEthernet1/0/4 service instance 1153
        Routes: 1 MAC, 1 MAC/IP
    Peers:
      1.1.1.3
        Routes: 0 MAC, 0 MAC/IP, 1 IMET, 0 EAD
```

### `show l2vpn evpn mac`

```
show l2vpn evpn mac
MAC Address    EVI   VLAN  ESI                      Ether Tag  Next Hop(s)
-------------- ----- ----- ------------------------ ---------- ---------------
5254.00c4.b4a1 1151  1151  0000.0000.0000.0000.0000 0          Gi1/0/3:1151
aabb.cc00.2400 1151  1151  0000.0000.0000.0000.0000 0          Gi1/0/4:1151
aabb.cc00.2700 1151  1151  0000.0000.0000.0000.0000 0          1.1.1.3
aabb.cc00.2400 1152  1152  0000.0000.0000.0000.0000 0          Gi1/0/4:1152
aabb.cc00.2700 1152  1152  0000.0000.0000.0000.0000 0          1.1.1.3
aabb.cc00.2400 1153  1153  0000.0000.0000.0000.0000 0          Gi1/0/4:1153
```

### `show mac address-table dynamic`

```
show mac address-table dynamic
          Mac Address Table
-------------------------------------------
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    aabb.cc00.2400    DYNAMIC     Gi1/0/4
1151    5254.00c4.b4a1    DYNAMIC     Gi1/0/3
1151    aabb.cc00.2400    DYNAMIC     Gi1/0/4
1152    aabb.cc00.2400    DYNAMIC     Gi1/0/4
1153    aabb.cc00.2400    DYNAMIC     Gi1/0/4
Total Mac Addresses for this criterion: 5
```

### `show ip route vrf VRF-1`

```
show ip route vrf VRF-1
Routing Table: VRF-1
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, m - OMP
       n - NAT, Ni - NAT inside, No - NAT outside, Nd - NAT DIA
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       H - NHRP, G - NHRP registered, g - NHRP registration summary
       o - ODR, P - periodic downloaded static route, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR
       & - replicated local route overrides by connected
Gateway of last resort is not set
      10.0.0.0/32 is subnetted, 3 subnets
B        10.1.1.1 [20/0] via 192.168.220.1, 01:24:27
B        10.1.1.2 [20/0] via 192.168.220.1, 01:24:27
B        10.1.1.3 [20/0] via 192.168.220.1, 01:24:27
      192.168.1.0/24 is variably subnetted, 3 subnets, 2 masks
C        192.168.1.0/25 is directly connected, Vlan1151
L        192.168.1.2/32 is directly connected, Vlan1151
B        192.168.1.23/32 [200/0] via 1.1.1.3, 01:23:18, Vlan1101
      192.168.2.0/24 is variably subnetted, 3 subnets, 2 masks
C        192.168.2.0/25 is directly connected, Vlan1152
L        192.168.2.2/32 is directly connected, Vlan1152
B        192.168.2.23/32 [200/0] via 1.1.1.3, 01:23:18, Vlan1101
      192.168.3.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.3.0/25 is directly connected, Vlan1153
L        192.168.3.2/32 is directly connected, Vlan1153
B     192.168.20.0/24 [20/0] via 192.168.220.1, 01:24:27
      192.168.220.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.220.0/24 is directly connected, GigabitEthernet1/0/5
L        192.168.220.2/32 is directly connected, GigabitEthernet1/0/5
B     192.168.221.0/24 [200/0] via 1.1.1.3, 01:11:51, Vlan1101
```

### `show ip arp vrf VRF-1`

```
show ip arp vrf VRF-1
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  1.1.1.2                 -   5254.0069.ac31  ARPA   Vlan1101
Internet  192.168.1.2             -   5254.0069.ac0a  ARPA   Vlan1151
Internet  192.168.2.3            52   5254.0018.447a  ARPA   Vlan1152
Internet  192.168.2.22           50   aabb.cc00.2400  ARPA   Vlan1152
Internet  192.168.2.23            0   aabb.cc00.2700  ARPA   Vlan1152
Internet  192.168.3.2             -   5254.0069.ac1b  ARPA   Vlan1153
Internet  192.168.3.3            60   5254.0018.4489  ARPA   Vlan1153
Internet  192.168.3.22           50   aabb.cc00.2400  ARPA   Vlan1153
Internet  192.168.220.1          85   aabb.cc00.2e00  ARPA   GigabitEthernet1/0/5
Internet  192.168.220.2           -   5254.0069.ac19  ARPA   GigabitEthernet1/0/5
```

## sw3

### `show nve peers`

```
show nve peers
'M' - MAC entry download flag  'A' - Adjacency download flag
'4' - IPv4 flag  '6' - IPv6 flag
Interface  VNI      Type Peer-IP          RMAC/Num_RTs   eVNI     state flags UP time
nve1       50101    L3CP 1.1.1.2          5254.0069.ac31 50101      UP  A/M/4 01:23:36
nve1       401151   L2CP 1.1.1.2          4              401151     UP   N/A  01:23:36
nve1       401152   L2CP 1.1.1.2          3              401152     UP   N/A  01:23:36
nve1       401153   L2CP 1.1.1.2          3              401153     UP   N/A  01:01:25
```

### `show nve vni`

```
show nve vni
Interface  VNI        Multicast-group  VNI state  Mode  VLAN  cfg vrf                      
nve1       401153     N/A              Up         L2CP  1153  CLI VRF-1                    
nve1       401152     N/A              Up         L2CP  1152  CLI VRF-1                    
nve1       401151     N/A              Up         L2CP  1151  CLI VRF-1                    
nve1       50101      N/A              Up         L3CP  1101  CLI VRF-1                    
```

### `show bgp l2vpn evpn summary`

```
show bgp l2vpn evpn summary
BGP router identifier 1.1.1.3, local AS number 6500
BGP table version is 62, main routing table version 62
39 network entries using 15288 bytes of memory
48 path entries using 11136 bytes of memory
20/19 BGP path/bestpath attribute entries using 5920 bytes of memory
1 BGP AS-PATH entries using 40 bytes of memory
12 BGP extended community entries using 564 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 32948 total bytes of memory
BGP activity 60/9 prefixes, 84/15 paths, scan interval 60 secs
39 networks peaked at 15:59:41 Feb 9 2026 UTC (00:35:49.668 ago)
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4         6500      98     110       62    0    0 01:24:34        0
1.1.1.2         4         6500     125     129       62    0    0 01:24:35       18
```

### `show bgp l2vpn evpn`

```
show bgp l2vpn evpn
BGP table version is 62, local router ID is 1.1.1.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
              t secondary path, L long-lived-stale,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found
     Network          Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1.1.1.2:1151
 *>i  [2][1.1.1.2:1151][0][48][525400C4B4A1][0][*]/20
                      1.1.1.2                  0    100      0 ?
 *>i  [2][1.1.1.2:1151][0][48][AABBCC002400][0][*]/20
                      1.1.1.2                  0    100      0 ?
 *>i  [2][1.1.1.2:1151][0][48][AABBCC002400][32][192.168.1.22]/24
                      1.1.1.2                  0    100      0 ?
Route Distinguisher: 1.1.1.2:1152
 *>i  [2][1.1.1.2:1152][0][48][AABBCC002400][0][*]/20
                      1.1.1.2                  0    100      0 ?
 *>i  [2][1.1.1.2:1152][0][48][AABBCC002400][32][192.168.2.22]/24
                      1.1.1.2                  0    100      0 ?
Route Distinguisher: 1.1.1.2:1153
 *>i  [2][1.1.1.2:1153][0][48][AABBCC002400][0][*]/20
                      1.1.1.2                  0    100      0 ?
 *>i  [2][1.1.1.2:1153][0][48][AABBCC002400][32][192.168.3.22]/24
                      1.1.1.2                  0    100      0 ?
Route Distinguisher: 1.1.1.3:1151
 *>i  [2][1.1.1.3:1151][0][48][525400C4B4A1][0][*]/20
                      1.1.1.2                  0    100      0 ?
 *>i  [2][1.1.1.3:1151][0][48][AABBCC002400][0][*]/20
                      1.1.1.2                  0    100      0 ?
 *>i  [2][1.1.1.3:1151][0][48][AABBCC002400][32][192.168.1.22]/24
                      1.1.1.2                  0    100      0 ?
 *>   [2][1.1.1.3:1151][0][48][AABBCC002700][0][*]/20
                      0.0.0.0                            32768 ?
 *>   [2][1.1.1.3:1151][0][48][AABBCC002700][32][192.168.1.23]/24
                      0.0.0.0                            32768 ?
Route Distinguisher: 1.1.1.3:1152
 *>i  [2][1.1.1.3:1152][0][48][AABBCC002400][0][*]/20
                      1.1.1.2                  0    100      0 ?
 *>i  [2][1.1.1.3:1152][0][48][AABBCC002400][32][192.168.2.22]/24
                      1.1.1.2                  0    100      0 ?
 *>   [2][1.1.1.3:1152][0][48][AABBCC002700][0][*]/20
                      0.0.0.0                            32768 ?
 *>   [2][1.1.1.3:1152][0][48][AABBCC002700][32][192.168.2.23]/24
                      0.0.0.0                            32768 ?
Route Distinguisher: 1.1.1.3:1153
 *>i  [2][1.1.1.3:1153][0][48][AABBCC002400][0][*]/20
                      1.1.1.2                  0    100      0 ?
 *>i  [2][1.1.1.3:1153][0][48][AABBCC002400][32][192.168.3.22]/24
                      1.1.1.2                  0    100      0 ?
Route Distinguisher: 1.1.1.3:11512
 *>i  [2][1.1.1.3:11512][0][48][AABBCC002400][0][*]/20
                      1.1.1.2                  0    100      0 ?
 *>i  [2][1.1.1.3:11512][0][48][AABBCC002400][32][192.168.2.22]/24
                      1.1.1.2                  0    100      0 ?
Route Distinguisher: 1.1.1.2:1151
 *>i  [3][1.1.1.2:1151][0][32][1.1.1.2]/17
                      1.1.1.2                  0    100      0 ?
Route Distinguisher: 1.1.1.2:1152
 *>i  [3][1.1.1.2:1152][0][32][1.1.1.2]/17
                      1.1.1.2                  0    100      0 ?
Route Distinguisher: 1.1.1.2:1153
 *>i  [3][1.1.1.2:1153][0][32][1.1.1.2]/17
                      1.1.1.2                  0    100      0 ?
Route Distinguisher: 1.1.1.3:1151
 *>i  [3][1.1.1.3:1151][0][32][1.1.1.2]/17
                      1.1.1.2                  0    100      0 ?
 *>   [3][1.1.1.3:1151][0][32][1.1.1.3]/17
                      0.0.0.0                            32768 ?
Route Distinguisher: 1.1.1.3:1152
 *>i  [3][1.1.1.3:1152][0][32][1.1.1.2]/17
                      1.1.1.2                  0    100      0 ?
 *>   [3][1.1.1.3:1152][0][32][1.1.1.3]/17
                      0.0.0.0                            32768 ?
Route Distinguisher: 1.1.1.3:1153
 *>i  [3][1.1.1.3:1153][0][32][1.1.1.2]/17
                      1.1.1.2                  0    100      0 ?
 *>   [3][1.1.1.3:1153][0][32][1.1.1.3]/17
                      0.0.0.0                            32768 ?
Route Distinguisher: 1.1.1.3:11512
 *>i  [3][1.1.1.3:11512][0][32][1.1.1.2]/17
                      1.1.1.2                  0    100      0 ?
Route Distinguisher: 10:1101 (default for vrf VRF-1)
 * i  [5][10:1101][0][24][192.168.20.0]/17
                      1.1.1.2                  0    100      0 100 6505 ?
 *>                    192.168.221.1            0             0 100 6505 ?
 *>i  [5][10:1101][0][24][192.168.220.0]/17
                      1.1.1.2                  0    100      0 ?
 *                     192.168.221.1            0             0 100 6505 ?
 *    [5][10:1101][0][24][192.168.221.0]/17
                      192.168.221.1            0             0 100 6505 ?
 *>                    0.0.0.0                  0         32768 ?
 * i  [5][10:1101][0][25][192.168.1.0]/17
                      1.1.1.2                  0    100      0 ?
 *>                    0.0.0.0                  0         32768 ?
 * i  [5][10:1101][0][25][192.168.2.0]/17
                      1.1.1.2                  0    100      0 ?
 *>                    0.0.0.0                  0         32768 ?
 *>   [5][10:1101][0][25][192.168.3.0]/17
                      0.0.0.0                  0         32768 ?
 * i                   1.1.1.2                  0    100      0 ?
 * i  [5][10:1101][0][32][10.1.1.1]/17
                      1.1.1.2                  0    100      0 100 6505 ?
 *>                    192.168.221.1            0             0 100 6505 ?
 * i  [5][10:1101][0][32][10.1.1.2]/17
                      1.1.1.2                  0    100      0 100 6505 ?
 *>                    192.168.221.1            0             0 100 6505 ?
 * i  [5][10:1101][0][32][10.1.1.3]/17
                      1.1.1.2                  0    100      0 100 6505 ?
 *>                    192.168.221.1            0             0 100 6505 ?
```

### `show l2vpn evpn evi detail`

```
show l2vpn evpn evi detail
EVPN instance:          1151 (VLAN Based)
  RD:                   1.1.1.3:1151 (auto)
  Import-RTs:           1151:1 6500:1151 
  Export-RTs:           1151:1 6500:1151 
  Per-EVI Label:        none
  State:                Established
  Replication Type:     Ingress (global)
  Encapsulation:        vxlan
  IP Local Learn:       Enabled (global)
  Adv. Def. Gateway:    Disabled (global)
  Re-originate RT5:     Disabled
  Adv. Multicast:       Disabled (global)
  AR Flood Suppress:    Enabled (global)
  Vlan:                 1151
    Protected:          False
    Ethernet-Tag:       0
    State:              Established
    Flood Suppress:     Attached
    Core If:            Vlan1101
    Access If:          Vlan1151
    NVE If:             nve1
    RMAC:               5254.0018.449f
    Core Vlan:          1101
    L2 VNI:             401151
    L3 VNI:             50101
    VTEP IP:            1.1.1.3
    Originating Router: 1.1.1.3
    VRF:                VRF-1
    IPv4 IRB:           Enabled (Local routing)
    IPv6 IRB:           Disabled
    Pseudoports:
      GigabitEthernet1/0/3 service instance 1151
        Routes: 0 MAC, 0 MAC/IP
      GigabitEthernet1/0/4 service instance 1151
        Routes: 1 MAC, 1 MAC/IP
    Peers:
      1.1.1.2
        Routes: 2 MAC, 1 MAC/IP, 1 IMET, 0 EAD
EVPN instance:          1152 (VLAN Based)
  RD:                   1.1.1.3:1152 (auto)
  Import-RTs:           6500:1152 
  Export-RTs:           6500:1152 
  Per-EVI Label:        none
  State:                Established
  Replication Type:     Ingress (global)
  Encapsulation:        vxlan
  IP Local Learn:       Enabled (global)
  Adv. Def. Gateway:    Disabled (global)
  Re-originate RT5:     Disabled
  Adv. Multicast:       Disabled (global)
  AR Flood Suppress:    Enabled (global)
  Vlan:                 1152
    Protected:          False
    Ethernet-Tag:       0
    State:              Established
    Flood Suppress:     Attached
    Core If:            Vlan1101
    Access If:          Vlan1152
    NVE If:             nve1
    RMAC:               5254.0018.449f
    Core Vlan:          1101
    L2 VNI:             401152
    L3 VNI:             50101
    VTEP IP:            1.1.1.3
    Originating Router: 1.1.1.3
    VRF:                VRF-1
    IPv4 IRB:           Enabled (Local routing)
    IPv6 IRB:           Disabled
    Pseudoports:
      GigabitEthernet1/0/4 service instance 1152
        Routes: 1 MAC, 1 MAC/IP
    Peers:
      1.1.1.2
        Routes: 1 MAC, 1 MAC/IP, 1 IMET, 0 EAD
EVPN instance:          1153 (VLAN Based)
  RD:                   1.1.1.3:1153 (auto)
  Import-RTs:           6500:1153 1153:1 
  Export-RTs:           6500:1153 1153:1 
  Per-EVI Label:        none
  State:                Established
  Replication Type:     Ingress (global)
  Encapsulation:        vxlan
  IP Local Learn:       Enabled (global)
  Adv. Def. Gateway:    Disabled (global)
  Re-originate RT5:     Disabled
  Adv. Multicast:       Disabled (global)
  AR Flood Suppress:    Enabled (global)
  Vlan:                 1153
    Protected:          False
    Ethernet-Tag:       0
    State:              Established
    Flood Suppress:     Attached
    Core If:            Vlan1101
    Access If:          Vlan1153
    NVE If:             nve1
    RMAC:               5254.0018.449f
    Core Vlan:          1101
    L2 VNI:             401153
    L3 VNI:             50101
    VTEP IP:            1.1.1.3
    Originating Router: 1.1.1.3
    VRF:                VRF-1
    IPv4 IRB:           Enabled (Local routing)
    IPv6 IRB:           Disabled
    Pseudoports:
      GigabitEthernet1/0/4 service instance 1153
        Routes: 0 MAC, 0 MAC/IP
    Peers:
      1.1.1.2
        Routes: 1 MAC, 1 MAC/IP, 1 IMET, 0 EAD
EVPN instance:          11512 (VLAN Based)
  RD:                   1.1.1.3:11512 (auto)
  Import-RTs:           1152:1 6500:11512 
  Export-RTs:           1152:1 6500:11512 
  Per-EVI Label:        none
  State:                Established
  Replication Type:     Ingress (global)
  Encapsulation:        vxlan
  IP Local Learn:       Enabled (global)
  Adv. Def. Gateway:    Disabled (global)
  Re-originate RT5:     Disabled
  Adv. Multicast:       Disabled (global)
  AR Flood Suppress:    Enabled (global)
```

### `show l2vpn evpn mac`

```
show l2vpn evpn mac
MAC Address    EVI   VLAN  ESI                      Ether Tag  Next Hop(s)
-------------- ----- ----- ------------------------ ---------- ---------------
5254.00c4.b4a1 1151  1151  0000.0000.0000.0000.0000 0          1.1.1.2
aabb.cc00.2400 1151  1151  0000.0000.0000.0000.0000 0          1.1.1.2
aabb.cc00.2700 1151  1151  0000.0000.0000.0000.0000 0          Gi1/0/4:1151
aabb.cc00.2400 1152  1152  0000.0000.0000.0000.0000 0          1.1.1.2
aabb.cc00.2700 1152  1152  0000.0000.0000.0000.0000 0          Gi1/0/4:1152
aabb.cc00.2400 1153  1153  0000.0000.0000.0000.0000 0          1.1.1.2
```

### `show mac address-table dynamic`

```
show mac address-table dynamic
          Mac Address Table
-------------------------------------------
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    aabb.cc00.2700    DYNAMIC     Gi1/0/4
1151    aabb.cc00.2700    DYNAMIC     Gi1/0/4
1152    aabb.cc00.2700    DYNAMIC     Gi1/0/4
Total Mac Addresses for this criterion: 3
```

### `show ip route vrf VRF-1`

```
show ip route vrf VRF-1
Routing Table: VRF-1
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, m - OMP
       n - NAT, Ni - NAT inside, No - NAT outside, Nd - NAT DIA
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       H - NHRP, G - NHRP registered, g - NHRP registration summary
       o - ODR, P - periodic downloaded static route, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR
       & - replicated local route overrides by connected
Gateway of last resort is not set
      10.0.0.0/32 is subnetted, 3 subnets
B        10.1.1.1 [20/0] via 192.168.221.1, 01:24:37
B        10.1.1.2 [20/0] via 192.168.221.1, 01:24:37
B        10.1.1.3 [20/0] via 192.168.221.1, 01:24:37
      192.168.1.0/24 is variably subnetted, 3 subnets, 2 masks
C        192.168.1.0/25 is directly connected, Vlan1151
L        192.168.1.3/32 is directly connected, Vlan1151
B        192.168.1.22/32 [200/0] via 1.1.1.2, 00:57:41, Vlan1101
      192.168.2.0/24 is variably subnetted, 3 subnets, 2 masks
C        192.168.2.0/25 is directly connected, Vlan1152
L        192.168.2.3/32 is directly connected, Vlan1152
B        192.168.2.22/32 [200/0] via 1.1.1.2, 01:24:00, Vlan1101
      192.168.3.0/24 is variably subnetted, 3 subnets, 2 masks
C        192.168.3.0/25 is directly connected, Vlan1153
L        192.168.3.3/32 is directly connected, Vlan1153
B        192.168.3.22/32 [200/0] via 1.1.1.2, 00:51:33, Vlan1101
B     192.168.20.0/24 [20/0] via 192.168.221.1, 01:24:37
B     192.168.220.0/24 [200/0] via 1.1.1.2, 01:24:00, Vlan1101
      192.168.221.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.221.0/24 is directly connected, GigabitEthernet1/0/5
L        192.168.221.2/32 is directly connected, GigabitEthernet1/0/5
```

### `show ip arp vrf VRF-1`

```
show ip arp vrf VRF-1
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  1.1.1.3                 -   5254.0018.449f  ARPA   Vlan1101
Internet  192.168.2.2            52   5254.0069.ac0c  ARPA   Vlan1152
Internet  192.168.2.22           51   aabb.cc00.2400  ARPA   Vlan1152
Internet  192.168.2.23            1   aabb.cc00.2700  ARPA   Vlan1152
Internet  192.168.3.2            51   aabb.cc00.2400  ARPA   Vlan1153
Internet  192.168.3.3             -   5254.0018.4489  ARPA   Vlan1153
Internet  192.168.221.1          84   aabb.cc00.2e10  ARPA   GigabitEthernet1/0/5
Internet  192.168.221.2           -   5254.0018.4487  ARPA   GigabitEthernet1/0/5
```


## Ping Tests (End-to-End Connectivity)

These tests verify Layer 2 and Layer 3 connectivity across the VXLAN fabric.

### Ping sw3 Loopback (underlay)

```
ping 1.1.1.3 source 1.1.1.2 repeat 3
Type escape sequence to abort.
Sending 3, 100-byte ICMP Echos to 1.1.1.3, timeout is 2 seconds:
Packet sent with a source address of 1.1.1.2 
!!!
Success rate is 100 percent (3/3), round-trip min/avg/max = 79/89/107 ms
```

### Ping remote host in VLAN 1151 (L2 extension)

```
ping vrf VRF-1 192.168.1.23 repeat 3
Type escape sequence to abort.
Sending 3, 100-byte ICMP Echos to 192.168.1.23, timeout is 2 seconds:
..
```

### Ping remote host in VLAN 1152 (L2 extension)

```
ping vrf VRF-1 192.168.2.23 repeat 3
Success rate is 0 percent (0/3)
```

### Ping External Router loopback (external BGP)

```
ping vrf VRF-1 10.1.1.1 repeat 3
Type escape sequence to abort.
Sending 3, 100-byte ICMP Echos to 10.1.1.1, timeout is 2 seconds:
!!!
Success rate is 100 percent (3/3), round-trip min/avg/max = 24/26/32 ms
```

