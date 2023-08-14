# Домашнее задание №7.
## VxLAN M-LAG. 
### Цель:
Настроить отказоустойчивое подключение клиентов с использованием MLAG 

### Решение.
Модифицируем лабораторную работу №6 для возможности отказоустойчивого подключения клиентов. Настройку произведем на оборудованиии Arista, поэтому развернем Multi-Chassis LAG (MLAG).

#### 1. Схема сети
![dz-7_topo_VxLAN_M-LAG.png](dz-7_topo_VxLAN_M-LAG.png)

#### 2. Адресное пространство

Адресация сетевых устройств:
|Name|Loopback0|Loopback1|Loopback1 sec|Eth-1|Eth-2|Eth-3|
|---|---|---|---|---|---|---|
Spine-1|10.1.0.1/32|N/A|N/A|10.1.5.0/31|10.1.5.2/31|10.1.5.4/31|
Spine-2|10.1.0.2/32|N/A|N/A|10.1.5.6/31|10.1.5.8/31|10.1.5.10/31|
Leaf-1|10.1.1.1/32|10.1.2.1/32|N/A|10.1.5.1/31|10.1.5.7/31|N/A|
Leaf-2|10.1.1.2/32|10.1.2.2/32|10.1.2.20/32|10.1.5.3/31|10.1.5.9/31|N/A|
Leaf-3|10.1.1.3/32|10.1.2.3/32|10.1.2.20/32|10.1.5.5/31|10.1.5.11/31|N/A|

Адресация клиентских устройств:
|Name|Client-1|Client-3|
|---|---|---|
IP/MASK|10.1.3.126/25|10.1.3.125/25|

#### 3. Настройки

Приведены настройки устройств, участвующих в работе MLAG.

##### Leaf-01
```
!
hostname Leaf-01
!
vlan 10
   name ###Service-1###
!
vrf instance SEMETIRB
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
   description ### Client-1 ###
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
   vxlan vlan 10-20 vni 10010-10020
   vxlan vrf SEMETIRB vni 65000
   vxlan vlan 10 flood vtep 10.1.2.20
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
      redistribute connected
!
```

##### Leaf-02 
```
!
hostname Leaf-02
!
vlan 10
   name ###Service-1###
!
vlan 20
   name ###Service-2###
!
vlan 4094
   name for_M-LAG
!
vrf instance SEMETIRB
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
   description ### Client-2 ###
   switchport access vlan 20
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
   ip address 10.1.1.2/32
!
interface Loopback1
   ip address 10.1.2.2/32
   ip address 10.1.2.20/32 secondary
!
interface Vlan20
   description ### Client-2 ###
   vrf SEMETIRB
   ip address virtual 10.1.3.129/25
!
interface Vlan4094
   description ### M-LAG peer-link ###
   ip address 192.168.0.1/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10-20 vni 10010-10020
   vxlan vrf SEMETIRB vni 65000
   vxlan vlan 10 flood vtep 10.1.2.1
!
ip virtual-router mac-address 00:11:22:33:44:55
!
ip routing
ip routing vrf SEMETIRB
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
      redistribute connected
!
```

##### Leaf-03
```
!
hostname Leaf-03
!
vlan 10
   name ###Service-1###
!
vlan 20
   name ###Service-2###
!
vlan 4094
   name for_M-LAG
!
vrf instance SEMETIRB
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
   description ### Client-1 ###
   switchport access vlan 10
!
interface Ethernet4
   description ### Client-2 ###
   switchport access vlan 20
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
   ip address 10.1.2.20/32 secondary
!
interface Vlan20
   description ### Client-2 ###
   vrf SEMETIRB
   ip address virtual 10.1.3.129/25
!
interface Vlan4094
   description ### M-LAG peer-link ###
   ip address 192.168.0.2/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10-20 vni 10010-10020
   vxlan vrf SEMETIRB vni 65000
   vxlan vlan 10 flood vtep 10.1.2.1
!
ip virtual-router mac-address 00:11:22:33:44:55
!
ip routing
ip routing vrf SEMETIRB
!
ip prefix-list LOOPBACKS seq 10 permit 10.1.0.0/22 le 32
!
mlag configuration
   domain-id Arista-1
   local-interface Vlan4094
   peer-address 192.168.0.1
   peer-link Port-Channel10
   dual-primary detection delay 5 action errdisable all-interfaces
   dual-primary recovery delay mlag 15 non-mlag 30
!
route-map LOOPBACKS permit 10
   match ip address prefix-list LOOPBACKS
!
router bgp 65003
   router-id 10.1.1.3
   neighbor SPINE_OVERLAY peer group
   neighbor SPINE_OVERLAY remote-as 65000
   neighbor SPINE_OVERLAY update-source Loopback0
   neighbor SPINE_OVERLAY ebgp-multihop 10
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
      redistribute connected
!
```

##### Client-01
```
NAME : VPCS[1]
IP/MASK : 10.1.3.126/25
GATEWAY : 10.1.3.1
```

##### Client-03
```
!
hostname Client-3
!
vlan 10
   name test_M-LAG
!
interface Port-Channel15
   switchport access vlan 10
!
interface Ethernet1
   switchport access vlan 10
   channel-group 15 mode active
!
interface Ethernet2
   switchport access vlan 10
   channel-group 15 mode active
!
interface Vlan10
   description ### test_M-LAG ###
   ip address 10.1.3.125/25
!
```

#### 4. Демонстрация работы VxLAN MLAG.
Проверим локальное взаимодействие Client-1 c Client-3 и наоборот.

##### Client-01
```
VPCS> ping 10.1.3.125
84 bytes from 10.1.3.125 icmp_seq=1 ttl=64 time=45.056 ms
84 bytes from 10.1.3.125 icmp_seq=2 ttl=64 time=26.370 ms
84 bytes from 10.1.3.125 icmp_seq=3 ttl=64 time=39.932 ms
84 bytes from 10.1.3.125 icmp_seq=4 ttl=64 time=29.254 ms
84 bytes from 10.1.3.125 icmp_seq=5 ttl=64 time=30.432 ms
```

##### Client-03
```
Client-3#ping 10.1.3.126
PING 10.1.3.126 (10.1.3.126) 72(100) bytes of data.
80 bytes from 10.1.3.126: icmp_seq=1 ttl=64 time=63.8 ms
80 bytes from 10.1.3.126: icmp_seq=2 ttl=64 time=60.4 ms
80 bytes from 10.1.3.126: icmp_seq=3 ttl=64 time=56.0 ms
80 bytes from 10.1.3.126: icmp_seq=4 ttl=64 time=53.4 ms
80 bytes from 10.1.3.126: icmp_seq=5 ttl=64 time=49.7 ms
--- 10.1.3.126 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 47ms
rtt min/avg/max/mdev = 49.713/56.723/63.857/5.005 ms, pipe 5, ipg/ewma 11.852/59.928 ms
```

##### Leaf-01

_вывод сокращен_
```
Leaf-01#show ip route
 B E      10.1.0.1/32 [200/0] via 10.1.5.0, Ethernet1
 B E      10.1.0.2/32 [200/0] via 10.1.5.6, Ethernet2
 C        10.1.1.1/32 is directly connected, Loopback0
 B E      10.1.1.2/32 [200/0] via 10.1.5.0, Ethernet1
                              via 10.1.5.6, Ethernet2
 B E      10.1.1.3/32 [200/0] via 10.1.5.0, Ethernet1
                              via 10.1.5.6, Ethernet2
 C        10.1.2.1/32 is directly connected, Loopback1
 B E      10.1.2.2/32 [200/0] via 10.1.5.0, Ethernet1
                              via 10.1.5.6, Ethernet2
 B E      10.1.2.3/32 [200/0] via 10.1.5.0, Ethernet1
                              via 10.1.5.6, Ethernet2
 B E      10.1.2.20/32 [200/0] via 10.1.5.0, Ethernet1
                               via 10.1.5.6, Ethernet2
 C        10.1.5.0/31 is directly connected, Ethernet1
 C        10.1.5.6/31 is directly connected, Ethernet2
```

```
Leaf-01#show mac address-table
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    0050.7966.680f    DYNAMIC     Et3        1       0:08:32 ago
  10    5000.00d0.a518    DYNAMIC     Vx1        2       0:00:57 ago
4094    5000.0087.d6b1    DYNAMIC     Vx1        1       2:28:50 ago
4094    5000.008c.5cd7    DYNAMIC     Vx1        1       5:16:54 ago
Total Mac Addresses for this criterion: 4
```

##### Leaf-02

```
Leaf-02#show mlag
MLAG Configuration:
domain-id                          :            Arista-1
local-interface                    :            Vlan4094
peer-address                       :         192.168.0.2
peer-link                          :      Port-Channel10
peer-config                        :          consistent

MLAG Status:
state                              :              Active
negotiation status                 :           Connected
peer-link status                   :                  Up
local-int status                   :                  Up
system-id                          :   52:00:00:87:d6:b1
dual-primary detection             :            Disabled
dual-primary interface errdisabled :               False

MLAG Ports:
Disabled                           :                   0
Configured                         :                   0
Inactive                           :                   0
Active-partial                     :                   0
Active-full                        :                   1
```

```
Leaf-02#show mlag interfaces detail
                                        local/remote
 mlag         state   local   remote    oper    config    last change   changes
------ ------------- ------- -------- ------- ---------- -------------- -------
   10   active-full    Po15     Po15   up/up   ena/ena    2:05:55 ago         8

```

```
Leaf-02#show mac address-table
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    0050.7966.680f    DYNAMIC     Vx1        1       0:10:43 ago
  10    5000.00d0.a518    DYNAMIC     Po15       1       0:10:43 ago
  20    5000.008c.5cd7    STATIC      Po10
4093    5000.008c.5cd7    DYNAMIC     Vx1        1       2:31:03 ago
4094    5000.008c.5cd7    STATIC      Po10
Total Mac Addresses for this criterion: 5
```

##### Leaf-03

```
Leaf-03#show mlag
MLAG Configuration:
domain-id                          :            Arista-1
local-interface                    :            Vlan4094
peer-address                       :         192.168.0.1
peer-link                          :      Port-Channel10
peer-config                        :          consistent

MLAG Status:
state                              :              Active
negotiation status                 :           Connected
peer-link status                   :                  Up
local-int status                   :                  Up
system-id                          :   52:00:00:87:d6:b1
dual-primary detection             :            Disabled
dual-primary interface errdisabled :               False

MLAG Ports:
Disabled                           :                   0
Configured                         :                   0
Inactive                           :                   0
Active-partial                     :                   0
Active-full                        :                   1
```

```
Leaf-03#show mlag interfaces detail
                                        local/remote
 mlag         state   local   remote    oper    config    last change   changes
------ ------------- ------- -------- ------- ---------- -------------- -------
   10   active-full    Po15     Po15   up/up   ena/ena    2:08:43 ago         7
```

```
Leaf-03#show mac address-table
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    0050.7966.680f    DYNAMIC     Vx1        1       0:13:27 ago
  10    5000.00d0.a518    DYNAMIC     Po15       1       0:13:28 ago
  20    5000.0087.d6b1    STATIC      Po10
4093    5000.0087.d6b1    DYNAMIC     Vx1        1       2:33:46 ago
4094    5000.0087.d6b1    STATIC      Po10
Total Mac Addresses for this criterion: 5
```

##### Иммитируем обрыв линка: 
```
Leaf-02#show interfaces status
Port       Name                    Status       Vlan     Duplex Speed  Type            Flags Encapsulation
Et1        ### to_Spine-1_eth2 ### connected    routed   full   10G    EbraTestPhyPort
Et2        ### to_Spine-2_eth2 ### connected    routed   full   10G    EbraTestPhyPort
Et3        ### Client-2 ###        connected    20       full   10G    EbraTestPhyPort
Et4                                connected    1        full   10G    EbraTestPhyPort
Et5        ### to_M-LAG_client ### notconnect   in Po15  full   10G    EbraTestPhyPort
Et6                                connected    1        full   10G    EbraTestPhyPort
Et7        ### Peer-link ###       connected    in Po10  full   10G    EbraTestPhyPort
Et8        ### Peer-link ###       connected    in Po10  full   10G    EbraTestPhyPort
Ma1                                connected    routed   a-full a-1G   10/100/1000
Po10       ### M-LAG peer-link ### connected    trunk    full   20G    N/A
Po15                               notconnect   10       full   10G    N/A
```

##### MAC-адрес пропал: 
```
Leaf-02#show mac address-table
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
  10    0050.7966.680f    DYNAMIC     Vx1        1       0:17:21 ago
  10    5000.00d0.a518    DYNAMIC     Po10       1       0:02:37 ago
  20    5000.008c.5cd7    STATIC      Po10
4093    5000.008c.5cd7    DYNAMIC     Vx1        1       2:37:41 ago
4094    5000.008c.5cd7    STATIC      Po10
Total Mac Addresses for this criterion: 5
```

##### Сlient-3 доступен потерь нет: 

```
84 bytes from 10.1.3.125 icmp_seq=1097 ttl=64 time=31.578 ms
84 bytes from 10.1.3.125 icmp_seq=1098 ttl=64 time=30.595 ms
84 bytes from 10.1.3.125 icmp_seq=1099 ttl=64 time=32.066 ms
84 bytes from 10.1.3.125 icmp_seq=1100 ttl=64 time=68.795 ms
84 bytes from 10.1.3.125 icmp_seq=1101 ttl=64 time=77.486 ms
84 bytes from 10.1.3.125 icmp_seq=1102 ttl=64 time=39.622 ms
84 bytes from 10.1.3.125 icmp_seq=1103 ttl=64 time=36.421 ms
84 bytes from 10.1.3.125 icmp_seq=1104 ttl=64 time=29.008 ms
84 bytes from 10.1.3.125 icmp_seq=1105 ttl=64 time=26.942 ms
84 bytes from 10.1.3.125 icmp_seq=1106 ttl=64 time=32.487 ms
84 bytes from 10.1.3.125 icmp_seq=1107 ttl=64 time=35.316 ms
84 bytes from 10.1.3.125 icmp_seq=1108 ttl=64 time=42.506 ms
84 bytes from 10.1.3.125 icmp_seq=1109 ttl=64 time=30.821 ms
84 bytes from 10.1.3.125 icmp_seq=1110 ttl=64 time=52.136 ms
84 bytes from 10.1.3.125 icmp_seq=1111 ttl=64 time=71.166 ms
84 bytes from 10.1.3.125 icmp_seq=1112 ttl=64 time=46.335 ms
84 bytes from 10.1.3.125 icmp_seq=1113 ttl=64 time=39.683 ms
84 bytes from 10.1.3.125 icmp_seq=1114 ttl=64 time=32.801 ms
```




