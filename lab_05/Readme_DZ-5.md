# Домашнее задание №4.
## Overlay. VxLAN EVPN L2. 
### Цель:
Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами

### Решение.
#### 1. Схема сети
Для демонстрации работы VxLAN EVPN L2 нанесем абонентские устройства на схему сети. 
![dz-5_topo_VxLAN_EVPN.png](dz-5_topo_VxLAN_EVPN.png)

#### 2. Адресное пространство
На Leaf устройствах добавим по одному Loopback интерфейсу, чтобы использовать в качестве источника VxLAN интерфейса.  

|Name|Loopback0|Loopback0|Eth-1|Eth-2|Eth-3|
|---|---|---|---|---|---|
Spine-1|10.1.0.1/32|N/A|10.1.5.0/31|10.1.5.2/31|10.1.5.4/31|
Spine-2|10.1.0.2/32|N/A|10.1.5.6/31|10.1.5.8/31|10.1.5.10/31|
Leaf-1|10.1.1.1/32|10.1.2.1/32|10.1.5.1/31|10.1.5.7/31|N/A|
Leaf-2|10.1.1.2/32|10.1.2.2/32|10.1.5.3/31|10.1.5.9/31|N/A|
Leaf-3|10.1.1.3/32|10.1.2.3/32|10.1.5.5/31|10.1.5.11/31|N/A|

#### 3. Настройки

##### Spine-01
```
!
service routing protocols model multi-agent
!
hostname Spine-01
!
interface Ethernet1
   description ### to_Leaf-1_eth1 ###
   no switchport
   ip address 10.1.5.0/31
!
interface Ethernet2
   description ### to_Leaf-2_eth1 ###
   no switchport
   ip address 10.1.5.2/31
!
interface Ethernet3
   description ### to_Leaf-3_eth1 ###
   no switchport
   ip address 10.1.5.4/31
!
interface Loopback0
   ip address 10.1.0.1/32
!
ip routing
!
ip prefix-list LOOPBACKS seq 10 permit 10.1.0.0/23 le 32
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
peer-filter LEAF_AS
   10 match as-range 65001-65003 result accept
!
router bgp 65000
   router-id 10.1.0.1
   bgp listen range 10.1.1.0/24 peer-group LEAF_OVERLAY peer-filter LEAF_AS
   bgp listen range 10.1.5.0/24 peer-group LEAF_UNDERLAY peer-filter LEAF_AS
   neighbor LEAF_OVERLAY peer group
   neighbor LEAF_OVERLAY update-source Loopback0
   neighbor LEAF_OVERLAY ebgp-multihop 2
   neighbor LEAF_OVERLAY send-community
   neighbor LEAF_UNDERLAY peer group
   redistribute connected route-map LOOPBACKS
   !
   address-family evpn
      neighbor LEAF_OVERLAY activate
   !
   address-family ipv4
      no neighbor LEAF_OVERLAY activate
!
```
##### Spine-02
```
!
service routing protocols model multi-agent
!
hostname Spine-02
!
interface Ethernet1
   description ### to_Leaf-1_eth2 ###
   no switchport
   ip address 10.1.5.6/31
!
interface Ethernet2
   description ### to_Leaf-2_eth2 ###
   no switchport
   ip address 10.1.5.8/31
!
interface Ethernet3
   description ### to_Leaf-3_eth2 ###
   no switchport
   ip address 10.1.5.10/31
!
interface Loopback0
   ip address 10.1.0.2/32
!
ip routing
!
ip prefix-list LOOPBACKS seq 10 permit 10.1.0.0/23 le 32
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
peer-filter LEAF_AS
   10 match as-range 65001-65003 result accept
!
router bgp 65000
   router-id 10.1.0.2
   bgp listen range 10.1.1.0/24 peer-group LEAF_OVERLAY peer-filter LEAF_AS
   bgp listen range 10.1.5.0/24 peer-group LEAF_UNDERLAY peer-filter LEAF_AS
   neighbor LEAF_OVERLAY peer group
   neighbor LEAF_OVERLAY update-source Loopback0
   neighbor LEAF_OVERLAY ebgp-multihop 2
   neighbor LEAF_OVERLAY send-community
   neighbor LEAF_UNDERLAY peer group
   redistribute connected route-map LOOPBACKS
   !
   address-family evpn
      neighbor LEAF_OVERLAY activate
   !
   address-family ipv4
      no neighbor LEAF_OVERLAY activate
!
```
##### Leaf-01
```
!
service routing protocols model multi-agent
!
hostname Leaf-01
!
spanning-tree mode mstp
!
vlan 10
   name ###Service-1###
!
interface Ethernet1
   description ### to_Spine-1_eth1 ###
   no switchport
   ip address 10.1.5.1/31
!
interface Ethernet2
   description ### to_Spine-2_eth1 ###
   no switchport
   ip address 10.1.5.7/31
!
interface Ethernet3
   switchport access vlan 10
!
interface Loopback0
   ip address 10.1.1.1/32
!
interface Loopback1
   ip address 10.1.2.1/32
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
!
ip prefix-list LOOPBACKS seq 10 permit 10.1.0.0/22 le 32
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
router bgp 65001
   router-id 10.1.1.1
   neighbor SPINE_OVERLA peer group
   neighbor SPINE_OVERLAY peer group
   neighbor SPINE_OVERLAY remote-as 65000
   neighbor SPINE_OVERLAY update-source Loopback0
   neighbor SPINE_OVERLAY ebgp-multihop 2
   neighbor SPINE_OVERLAY send-community
   neighbor SPINE_UNDERLAY peer group
   neighbor SPINE_UNDERLAY remote-as 65000
   neighbor 10.1.0.1 peer group SPINE_OVERLAY
   neighbor 10.1.0.2 peer group SPINE_OVERLAY
   neighbor 10.1.5.0 peer group SPINE_UNDERLAY
   neighbor 10.1.5.6 peer group SPINE_UNDERLAY
   redistribute connected route-map LOOPBACKS
   !
   vlan 10
      rd 10.1.1.1:10010
      route-target both 1:10010
      redistribute learned
   !
   address-family evpn
      neighbor SPINE_OVERLAY activate
   !
   address-family ipv4
      no neighbor SPINE_OVERLAY activate
!
```
##### Leaf-02 
```
!
service routing protocols model multi-agent
!
hostname Leaf-02
!
vlan 20
   name ###Service-2###
!
interface Ethernet1
   description ### to_Spine-1_eth2 ###
   no switchport
   ip address 10.1.5.3/31
!
interface Ethernet2
   description ### to_Spine-2_eth2 ###
   no switchport
   ip address 10.1.5.9/31
!
interface Ethernet3
   switchport access vlan 20
!
interface Loopback0
   ip address 10.1.1.2/32
!
interface Loopback1
   ip address 10.1.2.2/32
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 20 vni 10020
!
ip routing
!
ip prefix-list LOOPBACKS seq 10 permit 10.1.0.0/22 le 32
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
router bgp 65002
   router-id 10.1.1.2
   neighbor SPINE_OVERLAY peer group
   neighbor SPINE_OVERLAY remote-as 65000
   neighbor SPINE_OVERLAY update-source Loopback0
   neighbor SPINE_OVERLAY ebgp-multihop 2
   neighbor SPINE_OVERLAY send-community
   neighbor SPINE_UNDERLAY peer group
   neighbor SPINE_UNDERLAY remote-as 65000
   neighbor 10.1.0.1 peer group SPINE_OVERLAY
   neighbor 10.1.0.2 peer group SPINE_OVERLAY
   neighbor 10.1.5.2 peer group SPINE_UNDERLAY
   neighbor 10.1.5.8 peer group SPINE_UNDERLAY
   redistribute connected route-map LOOPBACKS
   !
   vlan 20
      rd 10.1.1.2:10020
      route-target both 1:10020
      redistribute learned
   !
   address-family evpn
      neighbor SPINE_OVERLAY activate
   !
   address-family ipv4
      no neighbor SPINE_OVERLAY activate
!
```
##### Leaf-03
```
!
service routing protocols model multi-agent
!
hostname Leaf-03
!
vlan 10
   name ###Service-1###
!
vlan 20
   name ###Service-2###
!
interface Ethernet1
   description ### to_Spine-1_eth3 ###
   no switchport
   ip address 10.1.5.5/31
!
interface Ethernet2
   description ### to_Spine-2_eth3 ###
   no switchport
   ip address 10.1.5.11/31
!
interface Ethernet3
   switchport access vlan 10
!
interface Ethernet4
   switchport access vlan 20
!
interface Loopback0
   ip address 10.1.1.3/32
!
interface Loopback1
   ip address 10.1.2.3/32
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10-20 vni 10010-10020
!
ip prefix-list LOOPBACKS seq 10 permit 10.1.0.0/22 le 32
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
router bgp 65003
   router-id 10.1.1.3
   neighbor SPINE_OVERLAY peer group
   neighbor SPINE_OVERLAY remote-as 65000
   neighbor SPINE_OVERLAY update-source Loopback0
   neighbor SPINE_OVERLAY ebgp-multihop 2
   neighbor SPINE_OVERLAY send-community
   neighbor SPINE_UNDERLAY peer group
   neighbor SPINE_UNDERLAY remote-as 65000
   neighbor 10.1.0.1 peer group SPINE_OVERLAY
   neighbor 10.1.0.2 peer group SPINE_OVERLAY
   neighbor 10.1.5.4 peer group SPINE_UNDERLAY
   neighbor 10.1.5.10 peer group SPINE_UNDERLAY
   redistribute connected route-map LOOPBACKS
   !
   vlan 10
      rd 10.1.1.3:10010
      route-target both 1:10010
      redistribute learned
   !
   vlan 20
      rd 10.1.1.3:10020
      route-target both 1:10020
      redistribute learned
   !
   address-family evpn
      neighbor SPINE_OVERLAY activate
   !
   address-family ipv4
      no neighbor SPINE_OVERLAY activate
!
```
##### Client-01
```
NAME        : VPCS[1]
IP/MASK     : 192.168.0.1/24
GATEWAY     : 192.168.0.254
```
##### Client-02
```
NAME        : VPCS[1]
IP/MASK     : 192.168.1.1/24
GATEWAY     : 192.168.1.254
```
##### Client-03
```
NAME        : VPCS[1]
IP/MASK     : 192.168.0.2/24
GATEWAY     : 192.168.0.254
```
##### Client-04
```
NAME        : VPCS[1]
IP/MASK     : 192.168.1.2/24
GATEWAY     : 192.168.1.254
```

#### 4. Демонстрация работы VxLAN EVPN L2. 

##### Spine-01
```
Spine-01#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.1.0.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor         V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.1         4 65001            301       292    0    0 03:43:01 Estab   1      1
  10.1.1.2         4 65002            286       286    0    0 03:42:26 Estab   1      1
  10.1.1.3         4 65003            279       262    0    0 03:35:11 Estab   2      2
```

```
Spine-01#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.1.0.1, local AS number 65000
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >     RD: 10.1.1.1:10010 imet 10.1.2.1
                                 10.1.2.1              -       100     0       65001 i
 * >     RD: 10.1.1.2:10020 imet 10.1.2.2
                                 10.1.2.2              -       100     0       65002 i
 * >     RD: 10.1.1.3:10010 imet 10.1.2.3
                                 10.1.2.3              -       100     0       65003 i
 * >     RD: 10.1.1.3:10020 imet 10.1.2.3
                                 10.1.2.3              -       100     0       65003 i
```
##### Spine-02
```
```

```
```
##### Leaf-01
```
```

```
```
##### Leaf-02 
```
```

```
```
##### Leaf-03
```
```

```
```
##### Client-01
```
```
##### Client-02
```
```
##### Client-03
```

```
##### Client-04
```
```