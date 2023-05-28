# Домашнее задание №2.
## Underlay. OSPF.
### Цель:
Настроить OSPF для Underlay сети

### Решение.
#### 1. Схема сети
![dz2_scheme_network](dz2_scheme_network.png)

#### 2. Адресное пространство

Примечание: _для достижения целей домашней работы и рационального распределения адресного пространства принято решение перейти на /31 маску физических интерфейсов._ 

|Name|Loopback0|Eth-1|Eth-2|Eth-3|
|---|---|---|---|---|
Spine-1|10.1.0.1/32|10.1.5.0/31|10.1.5.2/31|10.1.5.4/31|
Spine-2|10.1.0.2/32|10.1.5.6/31|10.1.5.8/31|10.1.5.10/31|
Leaf-1|10.1.1.1/32|10.1.5.1/31|10.1.5.7/31|N/A|
Leaf-2|10.1.1.2/32|10.1.5.3/31|10.1.5.9/31|N/A|
Leaf-3|10.1.1.3/32|10.1.5.5/31|10.1.5.11/31|N/A|

#### 3. Настройки

##### Spine-1
```
!
hostname Spine-1
!
interface Ethernet1
   description ### to_Leaf-1_eth1 ###
   no switchport
   ip address 10.1.5.0/31
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf authentication
   ip ospf authentication-key 7 7O4CGaiAcR0=
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description ### to_Leaf-2_eth1 ###
   no switchport
   ip address 10.1.5.2/31
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf authentication
   ip ospf authentication-key 7 x1W70kMMQaQ=
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description ### to_Leaf-3_eth1 ###
   no switchport
   ip address 10.1.5.4/31
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf authentication
   ip ospf authentication-key 7 x1W70kMMQaQ=
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.1.0.1/32
!
ip routing
!
router ospf 1
   router-id 10.1.0.1
   network 10.1.0.1/32 area 0.0.0.0
   max-lsa 12000
!
```
##### Spine-2
```
!
hostname Spine-2
!
interface Ethernet1
   description ### to_Leaf-1_eth2 ###
   no switchport
   ip address 10.1.5.6/31
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf authentication
   ip ospf authentication-key 7 7O4CGaiAcR0=
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description ### to_Leaf-2_eth2 ###
   no switchport
   ip address 10.1.5.8/31
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf authentication
   ip ospf authentication-key 7 x1W70kMMQaQ=
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description ### to_Leaf-3_eth2 ###
   no switchport
   ip address 10.1.5.10/31
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf authentication
   ip ospf authentication-key 7 x1W70kMMQaQ=
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.1.0.2/32
!
ip routing
!
router ospf 1
   router-id 10.1.0.2
   network 10.1.0.2/32 area 0.0.0.0
   max-lsa 12000
!
```
##### Leaf-1
```
!
hostname Leaf-1
!
interface Ethernet1
   description ### to_Spine-1_eth1 ###
   no switchport
   ip address 10.1.5.1/31
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf authentication
   ip ospf authentication-key 7 7O4CGaiAcR0=
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description ### to_Spine-2_eth1 ###
   no switchport
   ip address 10.1.5.7/31
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf authentication
   ip ospf authentication-key 7 x1W70kMMQaQ=
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.1.1.1/32
!
ip routing
!
router ospf 1
   router-id 10.1.1.1
   network 10.1.1.1/32 area 0.0.0.0
   max-lsa 12000
!
```
##### Leaf-2 
```
!
hostname Leaf-2
!
interface Ethernet1
   description ### to_Spine-1_eth2 ###
   no switchport
   ip address 10.1.5.3/31
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf authentication
   ip ospf authentication-key 7 7O4CGaiAcR0=
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description ### to_Spine-2_eth2 ###
   no switchport
   ip address 10.1.5.9/31
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf authentication
   ip ospf authentication-key 7 x1W70kMMQaQ=
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.1.1.2/32
!
ip routing
!
router ospf 1
   router-id 10.1.1.2
   network 10.1.1.2/32 area 0.0.0.0
   max-lsa 12000
!
```
##### Leaf-3
```
!
hostname Leaf-3
!
interface Ethernet1
   description ### to_Spine-1_eth3 ###
   no switchport
   ip address 10.1.5.5/31
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf authentication
   ip ospf authentication-key 7 7O4CGaiAcR0=
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description ### to_Spine-2_eth3 ###
   no switchport
   ip address 10.1.5.11/31
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf authentication
   ip ospf authentication-key 7 x1W70kMMQaQ=
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.1.1.3/32
!
ip routing
!
router ospf 1
   router-id 10.1.1.3
   network 10.1.1.3/32 area 0.0.0.0
   max-lsa 12000
!
```
#### 4. Демонстрация работы настроек протокола OSPF
##### Spine-1
```
Spine-1#show ip ospf neighbor
Neighbor ID     VRF      Pri State                  Dead Time   Address         Interface
10.1.1.1        default  0   FULL                   00:00:37    10.1.5.1        Ethernet1
10.1.1.2        default  0   FULL                   00:00:32    10.1.5.3        Ethernet2
10.1.1.3        default  0   FULL                   00:00:32    10.1.5.5        Ethernet3
Spine-1#
```
```
Spine-1#show ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - Dhcp client installed default route

Gateway of last resort is not set

 C      10.1.0.1/32 is directly connected, Loopback0
 O      10.1.0.2/32 [110/30] via 10.1.5.1, Ethernet1
                             via 10.1.5.3, Ethernet2
                             via 10.1.5.5, Ethernet3
 O      10.1.1.1/32 [110/20] via 10.1.5.1, Ethernet1
 O      10.1.1.2/32 [110/20] via 10.1.5.3, Ethernet2
 O      10.1.1.3/32 [110/20] via 10.1.5.5, Ethernet3
 C      10.1.5.0/31 is directly connected, Ethernet1
 C      10.1.5.2/31 is directly connected, Ethernet2
 C      10.1.5.4/31 is directly connected, Ethernet3
 O      10.1.5.6/31 [110/20] via 10.1.5.1, Ethernet1
 O      10.1.5.8/31 [110/20] via 10.1.5.3, Ethernet2
 O      10.1.5.10/31 [110/20] via 10.1.5.5, Ethernet3
```
```
Spine-1#show bfd neighbors
VRF name: default
-----------------
DstAddr      MyDisc   YourDisc     Interface    Type          LastUp  LastDown
-------- ---------- ---------- -------------- ------- --------------- ---------
10.1.5.1 2368358993 3757379659 Ethernet1(12)  normal  05/26/23 10:46        NA
10.1.5.3 1626724747 1556059404 Ethernet2(13)  normal  05/26/23 10:53        NA
10.1.5.5 3256435929 2207252392 Ethernet3(14)  normal  05/26/23 10:54        NA

        LastDiag    State
------------------- -----
   No Diagnostic       Up
   No Diagnostic       Up
   No Diagnostic       Up

```
##### Spine-2
```
Spine-2#show ip ospf neighbor
Neighbor ID     VRF      Pri State                  Dead Time   Address         Interface
10.1.1.1        default  0   FULL                   00:00:38    10.1.5.7        Ethernet1
10.1.1.2        default  0   FULL                   00:00:33    10.1.5.9        Ethernet2
10.1.1.3        default  0   FULL                   00:00:30    10.1.5.11       Ethernet3
```
```
Spine-2#show ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - Dhcp client installed default route

Gateway of last resort is not set

 O      10.1.0.1/32 [110/30] via 10.1.5.7, Ethernet1
                             via 10.1.5.9, Ethernet2
                             via 10.1.5.11, Ethernet3
 C      10.1.0.2/32 is directly connected, Loopback0
 O      10.1.1.1/32 [110/20] via 10.1.5.7, Ethernet1
 O      10.1.1.2/32 [110/20] via 10.1.5.9, Ethernet2
 O      10.1.1.3/32 [110/20] via 10.1.5.11, Ethernet3
 O      10.1.5.0/31 [110/20] via 10.1.5.7, Ethernet1
 O      10.1.5.2/31 [110/20] via 10.1.5.9, Ethernet2
 O      10.1.5.4/31 [110/20] via 10.1.5.11, Ethernet3
 C      10.1.5.6/31 is directly connected, Ethernet1
 C      10.1.5.8/31 is directly connected, Ethernet2
 C      10.1.5.10/31 is directly connected, Ethernet3
```
```
Spine-2#show bfd neighbors
VRF name: default
-----------------
DstAddr       MyDisc   YourDisc     Interface   Type          LastUp  LastDown
--------- ---------- ---------- ------------- ------- --------------- ---------
10.1.5.7  1549078110 1804904721 Ethernet1(12) normal  05/26/23 10:53        NA
10.1.5.9  1054505106 3500007661 Ethernet2(13) normal  05/26/23 10:54        NA
10.1.5.11 1378080671 3975481269 Ethernet3(14) normal  05/26/23 10:56        NA

        LastDiag    State
------------------- -----
   No Diagnostic       Up
   No Diagnostic       Up
   No Diagnostic       Up
```
##### Leaf-1
```
Leaf-1#show ip ospf neighbor
Neighbor ID     VRF      Pri State                  Dead Time   Address         Interface
10.1.0.1        default  0   FULL                   00:00:34    10.1.5.0        Ethernet1
10.1.0.2        default  0   FULL                   00:00:37    10.1.5.6        Ethernet2
```
```
Leaf-1#show ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - Dhcp client installed default route

Gateway of last resort is not set

 O      10.1.0.1/32 [110/20] via 10.1.5.0, Ethernet1
 O      10.1.0.2/32 [110/20] via 10.1.5.6, Ethernet2
 C      10.1.1.1/32 is directly connected, Loopback0
 O      10.1.1.2/32 [110/30] via 10.1.5.0, Ethernet1
                             via 10.1.5.6, Ethernet2
 O      10.1.1.3/32 [110/30] via 10.1.5.0, Ethernet1
                             via 10.1.5.6, Ethernet2
 C      10.1.5.0/31 is directly connected, Ethernet1
 O      10.1.5.2/31 [110/20] via 10.1.5.0, Ethernet1
 O      10.1.5.4/31 [110/20] via 10.1.5.0, Ethernet1
 C      10.1.5.6/31 is directly connected, Ethernet2
 O      10.1.5.8/31 [110/20] via 10.1.5.6, Ethernet2
 O      10.1.5.10/31 [110/20] via 10.1.5.6, Ethernet2
```
```
Leaf-1#show bfd neighbors
VRF name: default
-----------------
DstAddr      MyDisc   YourDisc     Interface    Type          LastUp  LastDown
-------- ---------- ---------- -------------- ------- --------------- ---------
10.1.5.0 3757379659 2368358993 Ethernet1(12)  normal  05/26/23 10:56        NA
10.1.5.6 1804904721 1549078110 Ethernet2(13)  normal  05/26/23 11:02        NA

        LastDiag    State
------------------- -----
   No Diagnostic       Up
   No Diagnostic       Up
```
##### Leaf-2 
```
Leaf-2#show ip ospf neighbor
Neighbor ID     VRF      Pri State                  Dead Time   Address         Interface
10.1.0.1        default  0   FULL                   00:00:31    10.1.5.2        Ethernet1
10.1.0.2        default  0   FULL                   00:00:30    10.1.5.8        Ethernet2
```
```
Leaf-2#show ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - Dhcp client installed default route

Gateway of last resort is not set

 O      10.1.0.1/32 [110/20] via 10.1.5.2, Ethernet1
 O      10.1.0.2/32 [110/20] via 10.1.5.8, Ethernet2
 O      10.1.1.1/32 [110/30] via 10.1.5.2, Ethernet1
                             via 10.1.5.8, Ethernet2
 C      10.1.1.2/32 is directly connected, Loopback0
 O      10.1.1.3/32 [110/30] via 10.1.5.2, Ethernet1
                             via 10.1.5.8, Ethernet2
 O      10.1.5.0/31 [110/20] via 10.1.5.2, Ethernet1
 C      10.1.5.2/31 is directly connected, Ethernet1
 O      10.1.5.4/31 [110/20] via 10.1.5.2, Ethernet1
 O      10.1.5.6/31 [110/20] via 10.1.5.8, Ethernet2
 C      10.1.5.8/31 is directly connected, Ethernet2
 O      10.1.5.10/31 [110/20] via 10.1.5.8, Ethernet2

```
```
Leaf-2#show bfd neighbors
VRF name: default
-----------------
DstAddr      MyDisc   YourDisc     Interface    Type          LastUp  LastDown
-------- ---------- ---------- -------------- ------- --------------- ---------
10.1.5.2 1556059404 1626724747 Ethernet1(12)  normal  05/26/23 10:52        NA
10.1.5.8 3500007661 1054505106 Ethernet2(13)  normal  05/26/23 10:53        NA

        LastDiag    State
------------------- -----
   No Diagnostic       Up
   No Diagnostic       Up
```
##### Leaf-3
```
```
