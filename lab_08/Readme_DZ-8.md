# Домашнее задание №8.
## EVPN route-type 5. 
### Цель:
Реализовать передачу суммарных префиксов через EVPN route-type 5 

### Решение.
Доработаем лабораторную работу №7 для возможности установки EVPN route-type 5 в таблицы маршрутизации VxLAN VRF. 

#### 1. Схема сети
![dz-8_topo_EVPN_rt-5.png](dz-8_topo_EVPN_rt-5.png)

#### 2. Адресное пространство

Адресация сетевых устройств:
|Name|Loopback0|Loopback1|Eth-1|Eth-2|Eth-3|
|---|---|---|---|---|---|
Spine-1|10.1.0.1/32|N/A|10.1.5.0/31|10.1.5.2/31|10.1.5.4/31|
Spine-2|10.1.0.2/32|N/A|10.1.5.6/31|10.1.5.8/31|10.1.5.10/31|
Leaf-1|10.1.1.1/32|10.1.2.1/32|10.1.5.1/31|10.1.5.7/31|N/A|
Leaf-2|10.1.1.2/32|10.1.2.2/32|10.1.5.3/31|10.1.5.9/31|N/A|
Leaf-3|10.1.1.3/32|10.1.2.3/32|10.1.5.5/31|10.1.5.11/31|N/A|

Адресация в VRF:
|Name|VRF-1|VRF-2|VRF-1|
|---|---|---|---|
Client NET|10.1.3.0/26|10.1.3.64/26|10.1.3.128/26|
p2p BR|172.16.1.0/30|172.16.2.0/30|172.16.3.0/30|

#### 3. Настройки
VxLAN фабрика реализована на eBGP для организации маршрутизации между VRF через BR роутер необходимо разрешить повторяющиеся номера AS в AS_PATH атрибуте.

##### Spine-01/02
```
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
peer-filter LEAF_AS
   10 match as-range 65001-65003 result accept
!
router bgp 65000
   router-id 10.1.0.1(2)
   bgp listen range 10.1.1.0/24 peer-group LEAF_OVERLAY peer-filter LEAF_AS
   bgp listen range 10.1.5.0/24 peer-group LEAF_UNDERLAY peer-filter LEAF_AS
   neighbor LEAF_OVERLAY peer group
   neighbor LEAF_OVERLAY update-source Loopback0
   neighbor LEAF_OVERLAY allowas-in 3
   neighbor LEAF_OVERLAY ebgp-multihop 2
   neighbor LEAF_OVERLAY send-community
   neighbor LEAF_UNDERLAY peer group
   neighbor LEAF_UNDERLAY allowas-in 3
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
hostname Leaf-01
!
vlan 10
   name _vrf-1_
!
vlan 11
   name _border-vrf-1_
!
vlan 20
   name _vrf-2_
!
vlan 22
   name _border-vrf-2_
!
vlan 30
   name _vrf-3_
!
vlan 33
   name _border-vrf-3_
!
vrf instance vrf-1
!
vrf instance vrf-2
!
vrf instance vrf-3
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
   no switchport
!
interface Ethernet3.11
   description ###_border-vrf-1_###
   encapsulation dot1q vlan 11
   vrf vrf-1
   ip address 172.16.1.1/30
!
interface Ethernet3.22
   description ###_border-vrf-2_###
   encapsulation dot1q vlan 22
   vrf vrf-2
   ip address 172.16.2.1/30
!
interface Ethernet3.33
   description ###_border-vrf-3_###
   encapsulation dot1q vlan 33
   vrf vrf-3
   ip address 172.16.3.1/30
!
interface Loopback0
   ip address 10.1.1.1/32
!
interface Loopback1
   ip address 10.1.2.1/32
!
interface Vlan10
   description ###_vrf-1_###
   vrf vrf-1
   ip address virtual 10.1.3.1/26
!
interface Vlan20
   description ###_vrf-2_###
   vrf vrf-2
   ip address virtual 10.1.3.65/26
!
interface Vlan30
   description ###_vrf-3_###
   vrf vrf-3
   ip address virtual 10.1.3.129/26
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vrf vrf-1 vni 1
   vxlan vrf vrf-2 vni 2
   vxlan vrf vrf-3 vni 3
!
ip virtual-router mac-address 00:11:22:33:44:55
!
ip routing
ip routing vrf vrf-1
ip routing vrf vrf-2
ip routing vrf vrf-3
!
ip prefix-list LOOPBACKS seq 10 permit 10.1.0.0/22 le 32
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
router bgp 65001
   router-id 10.1.1.1
   maximum-paths 128
   neighbor SPINE_OVERLA peer group
   neighbor SPINE_OVERLAY peer group
   neighbor SPINE_OVERLAY remote-as 65000
   neighbor SPINE_OVERLAY update-source Loopback0
   neighbor SPINE_OVERLAY ebgp-multihop 10
   neighbor SPINE_OVERLAY send-community
   neighbor SPINE_UNDERLAY peer group
   neighbor SPINE_UNDERLAY remote-as 65000
   neighbor 10.1.0.1 peer group SPINE_OVERLAY
   neighbor 10.1.0.2 peer group SPINE_OVERLAY
   neighbor 10.1.5.0 peer group SPINE_UNDERLAY
   neighbor 10.1.5.6 peer group SPINE_UNDERLAY
   redistribute connected route-map LOOPBACKS
   !
   address-family evpn
      neighbor SPINE_OVERLAY activate
   !
   address-family ipv4
      no neighbor SPINE_OVERLAY activate
   !
   vrf vrf-1
      rd 10.1.1.1:1
      route-target import evpn 1:1
      route-target export evpn 1:1
      neighbor 172.16.1.2 remote-as 4260232685
      neighbor 172.16.1.2 local-as 4259905537 no-prepend replace-as
      aggregate-address 10.1.3.0/26 summary-only
      redistribute connected
   !
   vrf vrf-2
      rd 10.1.1.1:2
      route-target import evpn 1:2
      route-target export evpn 1:2
      neighbor 172.16.2.2 remote-as 4260232685
      neighbor 172.16.2.2 local-as 4259905538 no-prepend replace-as
      aggregate-address 10.1.3.64/26 summary-only
      redistribute connected
   !
   vrf vrf-3
      rd 10.1.1.1:3
      route-target import evpn 1:3
      route-target export evpn 1:3
      neighbor 172.16.3.2 remote-as 4260232685
      neighbor 172.16.3.2 local-as 4259905539 no-prepend replace-as
      aggregate-address 10.1.3.128/26 summary-only
      redistribute connected
!
```

##### Leaf-02 
```
!
hostname Leaf-02
!
vlan 10
   name _vrf-1_
!
vlan 20
   name _vrf-2_
!
vlan 30
   name _vrf-3_
!
vlan 4094
   name for_M-LAG
!
vrf instance vrf-1
!
vrf instance vrf-2
!
vrf instance vrf-3
!
interface Port-Channel10
   description ### M-LAG peer-link ###
   switchport mode trunk
!
interface Port-Channel15
   description ###_vrf-1_###
   switchport access vlan 10
   mlag 10
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
   description ###_vrf-1_###
   switchport access vlan 10
!
interface Ethernet4
   description ###_vrf-2_###
   switchport access vlan 20
!
interface Ethernet5
   description ### to_M-LAG_client ###
   channel-group 15 mode active
!
interface Ethernet6
   description ###_vrf-3_###
   switchport access vlan 30
!
interface Ethernet7
   description ### Peer-link ###
   channel-group 10 mode active
!
interface Ethernet8
   description ### Peer-link ###
   channel-group 10 mode active
!
interface Loopback0
   ip address 10.1.1.2/32
!
interface Loopback1
   ip address 10.1.2.2/32
   ip address 10.1.2.20/32 secondary
!
interface Vlan10
   description ###_vrf-1_###
   vrf vrf-1
   ip address virtual 10.1.3.1/26
!
interface Vlan20
   description ###_vrf-2_###
   vrf vrf-2
   ip address virtual 10.1.3.65/26
!
interface Vlan30
   description ###_vrf-3_###
   vrf vrf-3
   ip address virtual 10.1.3.129/26
!
interface Vlan4094
   description ### M-LAG peer-link ###
   ip address 192.168.0.1/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10-20,30 vni 10010-10020,10030
   vxlan vrf vrf-1 vni 1
   vxlan vrf vrf-2 vni 2
   vxlan vrf vrf-3 vni 3
   vxlan vlan 10 flood vtep 10.1.2.1
!
ip virtual-router mac-address 00:11:22:33:44:55
!
ip routing
ip routing vrf vrf-1
ip routing vrf vrf-2
ip routing vrf vrf-3
!
ip prefix-list LOOPBACKS seq 10 permit 10.1.0.0/22 le 32
!
mlag configuration
   domain-id Arista-1
   local-interface Vlan4094
   peer-address 192.168.0.2
   peer-link Port-Channel10
   dual-primary detection delay 5 action errdisable all-interfaces
   dual-primary recovery delay mlag 15 non-mlag 30
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
router bgp 65002
   router-id 10.1.1.2
   neighbor SPINE_OVERLAY peer group
   neighbor SPINE_OVERLAY remote-as 65000
   neighbor SPINE_OVERLAY update-source Loopback0
   neighbor SPINE_OVERLAY allowas-in 3
   neighbor SPINE_OVERLAY ebgp-multihop 2
   neighbor SPINE_OVERLAY send-community
   neighbor SPINE_UNDERLAY peer group
   neighbor SPINE_UNDERLAY remote-as 65000
   neighbor SPINE_UNDERLAY allowas-in 3
   neighbor 10.1.0.1 peer group SPINE_OVERLAY
   neighbor 10.1.0.2 peer group SPINE_OVERLAY
   neighbor 10.1.5.2 peer group SPINE_UNDERLAY
   neighbor 10.1.5.8 peer group SPINE_UNDERLAY
   redistribute connected route-map LOOPBACKS
   !
   vlan 10
      rd 10.1.1.2:10010
      route-target both 1:10010
      redistribute learned
   !
   vlan 20
      rd 10.1.1.2:10020
      route-target both 1:10020
      redistribute learned
   !
   vlan 30
      rd 10.1.1.2:10030
      route-target both 1:10030
      redistribute learned
   !
   address-family evpn
      neighbor SPINE_OVERLAY activate
   !
   address-family ipv4
      no neighbor SPINE_OVERLAY activate
   !
   vrf vrf-1
      rd 10.1.1.2:1
      route-target import evpn 1:1
      route-target export evpn 1:1
      redistribute connected
   !
   vrf vrf-2
      rd 10.1.1.2:2
      route-target import evpn 1:2
      route-target export evpn 1:2
      redistribute connected
   !
   vrf vrf-3
      rd 10.1.1.2:3
      route-target import evpn 1:3
      route-target export evpn 1:3
      redistribute connected
!
```

##### Leaf-03
```
!
hostname Leaf-03
!
vlan 10
   name _vrf-1_
!
vlan 20
   name _vrf-2_
!
vlan 30
   name _vrf-3_
!
vlan 4094
   name for_M-LAG
!
vrf instance vrf-1
!
vrf instance vrf-2
!
vrf instance vrf-3
!
interface Port-Channel10
   description ### M-LAG peer-link ###
   switchport mode trunk
!
interface Port-Channel15
   switchport access vlan 10
   mlag 10
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
   description ###_vrf-2_###
   switchport access vlan 20
!
interface Ethernet4
   description ###_vrf-3_###
   switchport access vlan 30
!
interface Ethernet5
   description ### to_M-LAG_client ###
   channel-group 15 mode active
!
interface Ethernet7
   description ### Peer-link ###
   channel-group 10 mode active
!
interface Ethernet8
   description ### Peer-link ###
   channel-group 10 mode active
!
interface Loopback0
   ip address 10.1.1.3/32
!
interface Loopback1
   ip address 10.1.2.3/32
!
interface Vlan10
   description ###_vrf-1_###
   vrf vrf-1
   ip address virtual 10.1.3.1/26
!
interface Vlan20
   description ###_vrf-2_###
   vrf vrf-2
   ip address virtual 10.1.3.65/26
!
interface Vlan30
   description ###_vrf-3_###
   vrf vrf-3
   ip address virtual 10.1.3.129/26
!
interface Vlan4094
   description ### M-LAG peer-link ###
   ip address 192.168.0.2/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10-20,30 vni 10010-10020,10030
   vxlan vrf vrf-1 vni 1
   vxlan vrf vrf-2 vni 2
   vxlan vrf vrf-3 vni 3
   vxlan vlan 10 flood vtep 10.1.2.1
!
ip virtual-router mac-address 00:11:22:33:44:55
!
ip routing
ip routing vrf vrf-1
ip routing vrf vrf-2
ip routing vrf vrf-3
!
ip prefix-list LOOPBACKS seq 10 permit 10.1.0.0/22 le 32
!
mlag configuration
   domain-id Arista-1
   local-interface Vlan4094
   peer-address 192.168.0.1
   peer-link Port-Channel10
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
router bgp 65003
   router-id 10.1.1.3
   neighbor SPINE_OVERLAY peer group
   neighbor SPINE_OVERLAY remote-as 65000
   neighbor SPINE_OVERLAY update-source Loopback0
   neighbor SPINE_OVERLAY allowas-in 3
   neighbor SPINE_OVERLAY ebgp-multihop 10
   neighbor SPINE_OVERLAY send-community
   neighbor SPINE_UNDERLAY peer group
   neighbor SPINE_UNDERLAY remote-as 65000
   neighbor SPINE_UNDERLAY allowas-in 3
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
   vlan 30
      rd 10.1.1.3:10030
      route-target both 1:10030
      redistribute learned
   !
   address-family evpn
      neighbor SPINE_OVERLAY activate
   !
   address-family ipv4
      no neighbor SPINE_OVERLAY activate
   !
   vrf vrf-1
      rd 10.1.1.3:1
      route-target import evpn 1:1
      route-target export evpn 1:1
      redistribute connected
   !
   vrf vrf-2
      rd 10.1.1.3:2
      route-target import evpn 1:2
      route-target export evpn 1:2
      redistribute connected
   !
   vrf vrf-3
      rd 10.1.1.3:3
      route-target import evpn 1:3
      route-target export evpn 1:3
      redistribute connected
!
```

##### BR
```
!
hostname BR
!
interface Ethernet1
   no switchport
!
interface Ethernet1.11
   description ###_border-vrf-1_###
   encapsulation dot1q vlan 11
   ip address 172.16.1.2/30
!
interface Ethernet1.22
   description ###_border-vrf-2_###
   encapsulation dot1q vlan 22
   ip address 172.16.2.2/30
!
interface Ethernet1.33
   description ###_border-vrf-3_###
   encapsulation dot1q vlan 33
   ip address 172.16.3.2/30
!
interface Loopback0
   description ###_router-id_###
   ip address 10.1.0.254/32
!
interface Loopback1
   description ###_external_###
   ip address 8.8.8.8/32
!
ip routing
!
router bgp 4260232685
   neighbor 172.16.1.1 remote-as 4259905537
   neighbor 172.16.2.1 remote-as 4259905538
   neighbor 172.16.3.1 remote-as 4259905539
   aggregate-address 8.0.0.0/8 summary-only
   redistribute connected
!
```

#### 4. Демонстрация работы EVPN route-type 5
Убедимся, что на Leaf коммутаторах в BGP EVPN присутствуют маршруты типа 5

##### Leaf-01
```
Leaf-01#show bgp evpn route-type ip-prefix ipv4

          Network                Next Hop              Metric  LocPref Weight  Path
 * >     RD: 10.1.1.1:1 ip-prefix 8.0.0.0/8
                                 -                     -       100     0       4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 8.0.0.0/8
                                 -                     -       100     0       4260232685 i
 * >     RD: 10.1.1.1:3 ip-prefix 8.0.0.0/8
                                 -                     -       100     0       4260232685 i
 * >     RD: 10.1.1.1:1 ip-prefix 10.1.0.254/32
                                 -                     -       100     0       4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 10.1.0.254/32
                                 -                     -       100     0       4260232685 i
 * >     RD: 10.1.1.1:3 ip-prefix 10.1.0.254/32
                                 -                     -       100     0       4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 10.1.3.0/26
                                 -                     -       100     0       4260232685 4259905537 65000 65002 i
 * >     RD: 10.1.1.1:3 ip-prefix 10.1.3.0/26
                                 -                     -       100     0       4260232685 4259905537 65000 65002 i
 * >Ec   RD: 10.1.1.2:1 ip-prefix 10.1.3.0/26
                                 10.1.2.2              -       100     0       65000 65002 i
 *  ec   RD: 10.1.1.2:1 ip-prefix 10.1.3.0/26
                                 10.1.2.2              -       100     0       65000 65002 i
 * >Ec   RD: 10.1.1.3:1 ip-prefix 10.1.3.0/26
                                 10.1.2.3              -       100     0       65000 65003 i
 *  ec   RD: 10.1.1.3:1 ip-prefix 10.1.3.0/26
                                 10.1.2.3              -       100     0       65000 65003 i
 * >     RD: 10.1.1.1:1 ip-prefix 10.1.3.64/26
                                 -                     -       100     0       4260232685 4259905538 65000 65002 i
 * >     RD: 10.1.1.1:3 ip-prefix 10.1.3.64/26
                                 -                     -       100     0       4260232685 4259905538 65000 65002 i
 * >Ec   RD: 10.1.1.2:2 ip-prefix 10.1.3.64/26
                                 10.1.2.2              -       100     0       65000 65002 i
 *  ec   RD: 10.1.1.2:2 ip-prefix 10.1.3.64/26
                                 10.1.2.2              -       100     0       65000 65002 i
 * >Ec   RD: 10.1.1.3:2 ip-prefix 10.1.3.64/26
                                 10.1.2.3              -       100     0       65000 65003 i
 *  ec   RD: 10.1.1.3:2 ip-prefix 10.1.3.64/26
                                 10.1.2.3              -       100     0       65000 65003 i
 * >     RD: 10.1.1.1:1 ip-prefix 10.1.3.128/26
                                 -                     -       100     0       4260232685 4259905539 65000 65003 i
 * >     RD: 10.1.1.1:2 ip-prefix 10.1.3.128/26
                                 -                     -       100     0       4260232685 4259905539 65000 65003 i
 * >Ec   RD: 10.1.1.2:3 ip-prefix 10.1.3.128/26
                                 10.1.2.2              -       100     0       65000 65002 i
 *  ec   RD: 10.1.1.2:3 ip-prefix 10.1.3.128/26
                                 10.1.2.2              -       100     0       65000 65002 i
 * >Ec   RD: 10.1.1.3:3 ip-prefix 10.1.3.128/26
                                 10.1.2.3              -       100     0       65000 65003 i
 *  ec   RD: 10.1.1.3:3 ip-prefix 10.1.3.128/26
                                 10.1.2.3              -       100     0       65000 65003 i
 * >     RD: 10.1.1.1:1 ip-prefix 172.16.1.0/30
                                 -                     -       -       0       i
 *       RD: 10.1.1.1:1 ip-prefix 172.16.1.0/30
                                 -                     -       100     0       4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 172.16.1.0/30
                                 -                     -       100     0       4260232685 i
 * >     RD: 10.1.1.1:3 ip-prefix 172.16.1.0/30
                                 -                     -       100     0       4260232685 i
 * >     RD: 10.1.1.1:1 ip-prefix 172.16.2.0/30
                                 -                     -       100     0       4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 172.16.2.0/30
                                 -                     -       -       0       i
 *       RD: 10.1.1.1:2 ip-prefix 172.16.2.0/30
                                 -                     -       100     0       4260232685 i
 * >     RD: 10.1.1.1:3 ip-prefix 172.16.2.0/30
                                 -                     -       100     0       4260232685 i
 * >     RD: 10.1.1.1:1 ip-prefix 172.16.3.0/30
                                 -                     -       100     0       4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 172.16.3.0/30
                                 -                     -       100     0       4260232685 i
 * >     RD: 10.1.1.1:3 ip-prefix 172.16.3.0/30
                                 -                     -       -       0       i
 *       RD: 10.1.1.1:3 ip-prefix 172.16.3.0/30
                                 -                     -       100     0       4260232685 i
```

##### Leaf-02 
```
Leaf-02#show bgp evpn route-type ip-prefix ipv4

          Network                Next Hop              Metric  LocPref Weight  Path
 * >     RD: 10.1.1.1:1 ip-prefix 8.0.0.0/8
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:1 ip-prefix 8.0.0.0/8
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 8.0.0.0/8
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:2 ip-prefix 8.0.0.0/8
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:3 ip-prefix 8.0.0.0/8
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:3 ip-prefix 8.0.0.0/8
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:1 ip-prefix 10.1.0.254/32
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:1 ip-prefix 10.1.0.254/32
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 10.1.0.254/32
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:2 ip-prefix 10.1.0.254/32
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:3 ip-prefix 10.1.0.254/32
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:3 ip-prefix 10.1.0.254/32
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 10.1.3.0/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905537 65000 65002 i
 *       RD: 10.1.1.1:2 ip-prefix 10.1.3.0/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905537 65000 65002 i
 * >     RD: 10.1.1.1:3 ip-prefix 10.1.3.0/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905537 65000 65002 i
 *       RD: 10.1.1.1:3 ip-prefix 10.1.3.0/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905537 65000 65002 i
 * >     RD: 10.1.1.2:1 ip-prefix 10.1.3.0/26
                                 -                     -       -       0       i
 * >     RD: 10.1.1.3:1 ip-prefix 10.1.3.0/26
                                 10.1.2.3              -       100     0       65000 65003 i
 *       RD: 10.1.1.3:1 ip-prefix 10.1.3.0/26
                                 10.1.2.3              -       100     0       65000 65003 i
 * >     RD: 10.1.1.1:1 ip-prefix 10.1.3.64/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905538 65000 65002 i
 *       RD: 10.1.1.1:1 ip-prefix 10.1.3.64/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905538 65000 65002 i
 * >     RD: 10.1.1.1:3 ip-prefix 10.1.3.64/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905538 65000 65002 i
 *       RD: 10.1.1.1:3 ip-prefix 10.1.3.64/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905538 65000 65002 i
 * >     RD: 10.1.1.2:2 ip-prefix 10.1.3.64/26
                                 -                     -       -       0       i
 * >     RD: 10.1.1.3:2 ip-prefix 10.1.3.64/26
                                 10.1.2.3              -       100     0       65000 65003 i
 *       RD: 10.1.1.3:2 ip-prefix 10.1.3.64/26
                                 10.1.2.3              -       100     0       65000 65003 i
 * >     RD: 10.1.1.1:1 ip-prefix 10.1.3.128/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905539 65000 65003 i
 *       RD: 10.1.1.1:1 ip-prefix 10.1.3.128/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905539 65000 65003 i
 * >     RD: 10.1.1.1:2 ip-prefix 10.1.3.128/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905539 65000 65003 i
 *       RD: 10.1.1.1:2 ip-prefix 10.1.3.128/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905539 65000 65003 i
 * >     RD: 10.1.1.2:3 ip-prefix 10.1.3.128/26
                                 -                     -       -       0       i
 * >     RD: 10.1.1.3:3 ip-prefix 10.1.3.128/26
                                 10.1.2.3              -       100     0       65000 65003 i
 *       RD: 10.1.1.3:3 ip-prefix 10.1.3.128/26
                                 10.1.2.3              -       100     0       65000 65003 i
 * >     RD: 10.1.1.1:1 ip-prefix 172.16.1.0/30
                                 10.1.2.1              -       100     0       65000 65001 i
 *       RD: 10.1.1.1:1 ip-prefix 172.16.1.0/30
                                 10.1.2.1              -       100     0       65000 65001 i
 * >     RD: 10.1.1.1:2 ip-prefix 172.16.1.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:2 ip-prefix 172.16.1.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:3 ip-prefix 172.16.1.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:3 ip-prefix 172.16.1.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:1 ip-prefix 172.16.2.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:1 ip-prefix 172.16.2.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 172.16.2.0/30
                                 10.1.2.1              -       100     0       65000 65001 i
 *       RD: 10.1.1.1:2 ip-prefix 172.16.2.0/30
                                 10.1.2.1              -       100     0       65000 65001 i
 * >     RD: 10.1.1.1:3 ip-prefix 172.16.2.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:3 ip-prefix 172.16.2.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:1 ip-prefix 172.16.3.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:1 ip-prefix 172.16.3.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 172.16.3.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:2 ip-prefix 172.16.3.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:3 ip-prefix 172.16.3.0/30
                                 10.1.2.1              -       100     0       65000 65001 i
 *       RD: 10.1.1.1:3 ip-prefix 172.16.3.0/30
                                 10.1.2.1              -       100     0       65000 65001 i
```

##### Leaf-03
```
Leaf-03#show bgp evpn route-type ip-prefix ipv4

          Network                Next Hop              Metric  LocPref Weight  Path
 * >     RD: 10.1.1.1:1 ip-prefix 8.0.0.0/8
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:1 ip-prefix 8.0.0.0/8
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 8.0.0.0/8
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:2 ip-prefix 8.0.0.0/8
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:3 ip-prefix 8.0.0.0/8
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:3 ip-prefix 8.0.0.0/8
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:1 ip-prefix 10.1.0.254/32
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:1 ip-prefix 10.1.0.254/32
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 10.1.0.254/32
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:2 ip-prefix 10.1.0.254/32
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:3 ip-prefix 10.1.0.254/32
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:3 ip-prefix 10.1.0.254/32
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 10.1.3.0/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905537 65000 65002 i
 *       RD: 10.1.1.1:2 ip-prefix 10.1.3.0/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905537 65000 65002 i
 * >     RD: 10.1.1.1:3 ip-prefix 10.1.3.0/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905537 65000 65002 i
 *       RD: 10.1.1.1:3 ip-prefix 10.1.3.0/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905537 65000 65002 i
 * >     RD: 10.1.1.2:1 ip-prefix 10.1.3.0/26
                                 10.1.2.2              -       100     0       65000 65002 i
 *       RD: 10.1.1.2:1 ip-prefix 10.1.3.0/26
                                 10.1.2.2              -       100     0       65000 65002 i
 * >     RD: 10.1.1.3:1 ip-prefix 10.1.3.0/26
                                 -                     -       -       0       i
 * >     RD: 10.1.1.1:1 ip-prefix 10.1.3.64/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905538 65000 65002 i
 *       RD: 10.1.1.1:1 ip-prefix 10.1.3.64/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905538 65000 65002 i
 * >     RD: 10.1.1.1:3 ip-prefix 10.1.3.64/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905538 65000 65002 i
 *       RD: 10.1.1.1:3 ip-prefix 10.1.3.64/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905538 65000 65002 i
 * >     RD: 10.1.1.2:2 ip-prefix 10.1.3.64/26
                                 10.1.2.2              -       100     0       65000 65002 i
 *       RD: 10.1.1.2:2 ip-prefix 10.1.3.64/26
                                 10.1.2.2              -       100     0       65000 65002 i
 * >     RD: 10.1.1.3:2 ip-prefix 10.1.3.64/26
                                 -                     -       -       0       i
 * >     RD: 10.1.1.1:1 ip-prefix 10.1.3.128/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905539 65000 65003 i
 *       RD: 10.1.1.1:1 ip-prefix 10.1.3.128/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905539 65000 65003 i
 * >     RD: 10.1.1.1:2 ip-prefix 10.1.3.128/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905539 65000 65003 i
 *       RD: 10.1.1.1:2 ip-prefix 10.1.3.128/26
                                 10.1.2.1              -       100     0       65000 65001 4260232685 4259905539 65000 65003 i
 * >     RD: 10.1.1.2:3 ip-prefix 10.1.3.128/26
                                 10.1.2.2              -       100     0       65000 65002 i
 *       RD: 10.1.1.2:3 ip-prefix 10.1.3.128/26
                                 10.1.2.2              -       100     0       65000 65002 i
 * >     RD: 10.1.1.3:3 ip-prefix 10.1.3.128/26
                                 -                     -       -       0       i
 * >     RD: 10.1.1.1:1 ip-prefix 172.16.1.0/30
                                 10.1.2.1              -       100     0       65000 65001 i
 *       RD: 10.1.1.1:1 ip-prefix 172.16.1.0/30
                                 10.1.2.1              -       100     0       65000 65001 i
 * >     RD: 10.1.1.1:2 ip-prefix 172.16.1.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:2 ip-prefix 172.16.1.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:3 ip-prefix 172.16.1.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:3 ip-prefix 172.16.1.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:1 ip-prefix 172.16.2.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:1 ip-prefix 172.16.2.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 172.16.2.0/30
                                 10.1.2.1              -       100     0       65000 65001 i
 *       RD: 10.1.1.1:2 ip-prefix 172.16.2.0/30
                                 10.1.2.1              -       100     0       65000 65001 i
 * >     RD: 10.1.1.1:3 ip-prefix 172.16.2.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:3 ip-prefix 172.16.2.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:1 ip-prefix 172.16.3.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:1 ip-prefix 172.16.3.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:2 ip-prefix 172.16.3.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 *       RD: 10.1.1.1:2 ip-prefix 172.16.3.0/30
                                 10.1.2.1              -       100     0       65000 65001 4260232685 i
 * >     RD: 10.1.1.1:3 ip-prefix 172.16.3.0/30
                                 10.1.2.1              -       100     0       65000 65001 i
 *       RD: 10.1.1.1:3 ip-prefix 172.16.3.0/30
                                 10.1.2.1              -       100     0       65000 65001 i
```

Рассмотрим таблицы маршрутизации VRF. Убедимся, что агрегированный префикс c BR и сети других VRF устанавливаются. 

##### Leaf-01
```
Leaf-01#show ip route vrf vrf-1

VRF: vrf-1

 B E      8.0.0.0/8 [200/0] via 172.16.1.2, Ethernet3.11
 B E      10.1.0.254/32 [200/0] via 172.16.1.2, Ethernet3.11
 B E      10.1.3.0/26 [200/0] via VTEP 10.1.2.3 VNI 1 router-mac 50:00:00:8c:5c:d7 local-interface Vxlan1
                              via VTEP 10.1.2.2 VNI 1 router-mac 50:00:00:87:d6:b1 local-interface Vxlan1
 B E      10.1.3.64/26 [200/0] via 172.16.1.2, Ethernet3.11
 B E      10.1.3.128/26 [200/0] via 172.16.1.2, Ethernet3.11
 C        172.16.1.0/30 is directly connected, Ethernet3.11
 B E      172.16.2.0/30 [200/0] via 172.16.1.2, Ethernet3.11
 B E      172.16.3.0/30 [200/0] via 172.16.1.2, Ethernet3.11


Leaf-01#show ip route vrf vrf-2

VRF: vrf-2

 B E      8.0.0.0/8 [200/0] via 172.16.2.2, Ethernet3.22
 B E      10.1.0.254/32 [200/0] via 172.16.2.2, Ethernet3.22
 B E      10.1.3.0/26 [200/0] via 172.16.2.2, Ethernet3.22
 B E      10.1.3.64/26 [200/0] via VTEP 10.1.2.3 VNI 2 router-mac 50:00:00:8c:5c:d7 local-interface Vxlan1
                               via VTEP 10.1.2.2 VNI 2 router-mac 50:00:00:87:d6:b1 local-interface Vxlan1
 B E      10.1.3.128/26 [200/0] via 172.16.2.2, Ethernet3.22
 B E      172.16.1.0/30 [200/0] via 172.16.2.2, Ethernet3.22
 C        172.16.2.0/30 is directly connected, Ethernet3.22
 B E      172.16.3.0/30 [200/0] via 172.16.2.2, Ethernet3.22


Leaf-01#show ip route vrf vrf-3

VRF: vrf-3

 B E      8.0.0.0/8 [200/0] via 172.16.3.2, Ethernet3.33
 B E      10.1.0.254/32 [200/0] via 172.16.3.2, Ethernet3.33
 B E      10.1.3.0/26 [200/0] via 172.16.3.2, Ethernet3.33
 B E      10.1.3.64/26 [200/0] via 172.16.3.2, Ethernet3.33
 B E      10.1.3.128/26 [200/0] via VTEP 10.1.2.3 VNI 3 router-mac 50:00:00:8c:5c:d7 local-interface Vxlan1
                                via VTEP 10.1.2.2 VNI 3 router-mac 50:00:00:87:d6:b1 local-interface Vxlan1
 B E      172.16.1.0/30 [200/0] via 172.16.3.2, Ethernet3.33
 B E      172.16.2.0/30 [200/0] via 172.16.3.2, Ethernet3.33
 C        172.16.3.0/30 is directly connected, Ethernet3.33
```

##### Leaf-02 
```
Leaf-02#show ip route vrf vrf-1

VRF: vrf-1

 B E      8.0.0.0/8 [200/0] via VTEP 10.1.2.1 VNI 1 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      10.1.0.254/32 [200/0] via VTEP 10.1.2.1 VNI 1 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 C        10.1.3.0/26 is directly connected, Vlan10
 B E      10.1.3.64/26 [200/0] via VTEP 10.1.2.1 VNI 1 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      10.1.3.128/26 [200/0] via VTEP 10.1.2.1 VNI 1 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.1.0/30 [200/0] via VTEP 10.1.2.1 VNI 1 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.2.0/30 [200/0] via VTEP 10.1.2.1 VNI 1 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.3.0/30 [200/0] via VTEP 10.1.2.1 VNI 1 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1

Leaf-02#show ip route vrf vrf-2

VRF: vrf-2

 B E      8.0.0.0/8 [200/0] via VTEP 10.1.2.1 VNI 2 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      10.1.0.254/32 [200/0] via VTEP 10.1.2.1 VNI 2 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      10.1.3.0/26 [200/0] via VTEP 10.1.2.1 VNI 2 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 C        10.1.3.64/26 is directly connected, Vlan20
 B E      10.1.3.128/26 [200/0] via VTEP 10.1.2.1 VNI 2 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.1.0/30 [200/0] via VTEP 10.1.2.1 VNI 2 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.2.0/30 [200/0] via VTEP 10.1.2.1 VNI 2 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.3.0/30 [200/0] via VTEP 10.1.2.1 VNI 2 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1

Leaf-02#show ip route vrf vrf-3

VRF: vrf-3

 B E      8.0.0.0/8 [200/0] via VTEP 10.1.2.1 VNI 3 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      10.1.0.254/32 [200/0] via VTEP 10.1.2.1 VNI 3 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      10.1.3.0/26 [200/0] via VTEP 10.1.2.1 VNI 3 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      10.1.3.64/26 [200/0] via VTEP 10.1.2.1 VNI 3 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 C        10.1.3.128/26 is directly connected, Vlan30
 B E      172.16.1.0/30 [200/0] via VTEP 10.1.2.1 VNI 3 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.2.0/30 [200/0] via VTEP 10.1.2.1 VNI 3 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.3.0/30 [200/0] via VTEP 10.1.2.1 VNI 3 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
```

##### Leaf-03
```
Leaf-03#show ip route vrf vrf-1

VRF: vrf-1

 B E      8.0.0.0/8 [200/0] via VTEP 10.1.2.1 VNI 1 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      10.1.0.254/32 [200/0] via VTEP 10.1.2.1 VNI 1 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 C        10.1.3.0/26 is directly connected, Vlan10
 B E      10.1.3.64/26 [200/0] via VTEP 10.1.2.1 VNI 1 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      10.1.3.128/26 [200/0] via VTEP 10.1.2.1 VNI 1 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.1.0/30 [200/0] via VTEP 10.1.2.1 VNI 1 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.2.0/30 [200/0] via VTEP 10.1.2.1 VNI 1 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.3.0/30 [200/0] via VTEP 10.1.2.1 VNI 1 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1

Leaf-03#show ip route vrf vrf-2

VRF: vrf-2

 B E      8.0.0.0/8 [200/0] via VTEP 10.1.2.1 VNI 2 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      10.1.0.254/32 [200/0] via VTEP 10.1.2.1 VNI 2 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      10.1.3.0/26 [200/0] via VTEP 10.1.2.1 VNI 2 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 C        10.1.3.64/26 is directly connected, Vlan20
 B E      10.1.3.128/26 [200/0] via VTEP 10.1.2.1 VNI 2 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.1.0/30 [200/0] via VTEP 10.1.2.1 VNI 2 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.2.0/30 [200/0] via VTEP 10.1.2.1 VNI 2 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.3.0/30 [200/0] via VTEP 10.1.2.1 VNI 2 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1

Leaf-03#show ip route vrf vrf-3

VRF: vrf-3

 B E      8.0.0.0/8 [200/0] via VTEP 10.1.2.1 VNI 3 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      10.1.0.254/32 [200/0] via VTEP 10.1.2.1 VNI 3 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      10.1.3.0/26 [200/0] via VTEP 10.1.2.1 VNI 3 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      10.1.3.64/26 [200/0] via VTEP 10.1.2.1 VNI 3 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 C        10.1.3.128/26 is directly connected, Vlan30
 B E      172.16.1.0/30 [200/0] via VTEP 10.1.2.1 VNI 3 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.2.0/30 [200/0] via VTEP 10.1.2.1 VNI 3 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
 B E      172.16.3.0/30 [200/0] via VTEP 10.1.2.1 VNI 3 router-mac 50:00:00:a1:7a:a7 local-interface Vxlan1
```

Убедимся, что маршрутизация с клиентских устройств осуществляется через BR

Trace между VRF
```
PC-6> trace 10.1.3.2
trace to 10.1.3.2, 8 hops max, press Ctrl+C to stop
 1   10.1.3.129   5.564 ms  3.920 ms  4.557 ms
 2   172.16.3.1   25.991 ms  21.684 ms  22.024 ms
 3   172.16.3.2   30.754 ms  22.397 ms  21.036 ms
 4   172.16.1.1   29.543 ms  28.780 ms  34.205 ms
 5   10.1.3.1   56.250 ms  56.008 ms  56.409 ms
 6   *10.1.3.2   476.255 ms (ICMP type:3, code:3, Destination port unreachable)  313.949 ms  *
 7   10.1.3.2   459.266 ms  297.481 ms  *
 8   10.1.3.2   801.373 ms  *  363.146 ms
```
Trace внутри VRF 
```
PC-6> trace 10.1.3.130
trace to 10.1.3.130, 8 hops max, press Ctrl+C to stop
 1   *10.1.3.130   117.505 ms (ICMP type:3, code:3, Destination port unreachable)
```

Таким образом, цели домашней работы достигнуты EVPN route-type 5 передаются.

