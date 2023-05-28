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
