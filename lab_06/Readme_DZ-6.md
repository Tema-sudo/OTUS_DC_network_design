# Домашнее задание №6.
## Overlay. VxLAN EVPN L3. 
### Цель:
Настроить маршрутизацию в рамках Overlay между клиентами

### Решение.
Доработаем лабораторную работу №5 для организации маршрутизации между клиентами в разных подсетях. 

#### 1. Схема сети
![dz-6_topo_VxLAN_EVPNL3.png](dz-6_topo_VxLAN_EVPNL3.png)

#### 2. Адресное пространство

Адресация сетевых устройств:
|Name|Loopback0|Loopback1|Eth-1|Eth-2|Eth-3|
|---|---|---|---|---|---|
Spine-1|10.1.0.1/32|N/A|10.1.5.0/31|10.1.5.2/31|10.1.5.4/31|
Spine-2|10.1.0.2/32|N/A|10.1.5.6/31|10.1.5.8/31|10.1.5.10/31|
Leaf-1|10.1.1.1/32|10.1.2.1/32|10.1.5.1/31|10.1.5.7/31|N/A|
Leaf-2|10.1.1.2/32|10.1.2.2/32|10.1.5.3/31|10.1.5.9/31|N/A|
Leaf-3|10.1.1.3/32|10.1.2.3/32|10.1.5.5/31|10.1.5.11/31|N/A|

Адресация клиентских устройств:
|Name|Client-1|Client-2|Client-3|Client-4|
|---|---|---|---|---|
IP/MASK|10.1.3.126/25|10.1.3.254/25|10.1.3.125/25|10.1.3.253/25|
GW|10.1.3.1|10.1.3.129|10.1.3.1|10.1.5.8/31|10.1.3.129|

#### 3. Настройки

Так как в качестве базовой конфигурации используется лабораторная работа №5, ниже приведены донастройки для организации маршрутизации Overlay VxLAN EVPN L3.

##### Leaf-01
```
!
vrf instance SEMETIRB
!
interface Vlan10
   description ### Client-1 ###
   vrf SEMETIRB
   ip address 10.1.3.1/25
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vrf SEMETIRB vni 65000
!
ip virtual-router mac-address 00:11:22:33:44:55
!
ip routing
ip routing vrf SEMETIRB
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
   vrf SEMETIRB
      rd 10.1.1.1:65000
      route-target import evpn 1:65000
      route-target export evpn 1:65000
!
```

##### Leaf-02 
```
!
vrf instance SEMETIRB
!
interface Vlan20
   description ### Client-2 ###
   vrf SEMETIRB
   ip address 10.1.3.129/25
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 20 vni 10020
   vxlan vrf SEMETIRB vni 65000
!
ip virtual-router mac-address 00:11:22:33:44:55
!
ip routing
ip routing vrf SEMETIRB
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
   vrf SEMETIRB
      rd 10.1.1.2:65000
      route-target import evpn 1:65000
      route-target export evpn 1:65000
!
```

##### Leaf-03
```
!
vrf instance SEMETIRB
!
interface Vlan10
   description ### Client-1 ###
   vrf SEMETIRB
   ip address 10.1.3.1/25
!
interface Vlan20
   description ### Client-2 ###
   vrf SEMETIRB
   ip address 10.1.3.129/25
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10-20 vni 10010-10020
   vxlan vrf SEMETIRB vni 65000
!
ip virtual-router mac-address 00:11:22:33:44:55
!
ip routing
ip routing vrf SEMETIRB
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
   vrf SEMETIRB
      rd 10.1.1.3:65000
      route-target import evpn 1:65000
      route-target export evpn 1:65000
!
```

##### Client-01
```
NAME : VPCS[1]
IP/MASK : 10.1.3.126/25
GATEWAY : 10.1.3.1
```

##### Client-02
```
NAME : VPCS[1]
IP/MASK : 10.1.3.254/25
GATEWAY : 10.1.3.129
```

##### Client-03
```
NAME : VPCS[1]
IP/MASK : 10.1.3.125/25
GATEWAY : 10.1.3.1
```

##### Client-04
```
NAME : VPCS[1]
IP/MASK : 10.1.3.253/25
GATEWAY : 10.1.3.129
```