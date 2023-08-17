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

```

