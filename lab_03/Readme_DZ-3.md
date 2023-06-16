# Домашнее задание №3.
## Underlay. IS-IS.
### Цель:
Настроить IS-IS для Underlay сети

### Решение.
#### 1. Схема сети

Для Underlay сети реализуем IS-IS Level 2 топологию.

![dz-3_topo_isis_clos.png](dz-3_topo_isis_clos.png)

#### 2. Адресное пространство

Используем /31 маску для организации адресного простраснтва

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
   isis enable OTUS
   isis network point-to-point
   isis authentication mode text
   isis authentication key 7 Y5rbfDnzn74=
!
interface Ethernet2
   description ### to_Leaf-2_eth1 ###
   no switchport
   ip address 10.1.5.2/31
   isis enable OTUS
   isis network point-to-point
   isis authentication mode text
   isis authentication key 7 Y5rbfDnzn74=
!
interface Ethernet3
   description ### to_Leaf-3_eth1 ###
   no switchport
   ip address 10.1.5.4/31
   isis enable OTUS
   isis network point-to-point
   isis authentication mode text
   isis authentication key 7 Y5rbfDnzn74=
!
interface Loopback0
   ip address 10.1.0.1/32
   isis enable OTUS
!
ip routing
!
router isis OTUS
   net 49.0011.0100.0100.0001.00
   is-type level-2
   !
   address-family ipv4 unicast
      bfd all-interfaces
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
   isis enable OTUS
   isis network point-to-point
   isis authentication mode text
   isis authentication key 7 Y5rbfDnzn74=
!
interface Ethernet2
   description ### to_Leaf-2_eth2 ###
   no switchport
   ip address 10.1.5.8/31
   isis enable OTUS
   isis network point-to-point
   isis authentication mode text
   isis authentication key 7 Y5rbfDnzn74=
!
interface Ethernet3
   description ### to_Leaf-3_eth2 ###
   no switchport
   ip address 10.1.5.10/31
   isis enable OTUS
   isis network point-to-point
   isis authentication mode text
   isis authentication key 7 Y5rbfDnzn74=
!
interface Loopback0
   ip address 10.1.0.2/32
   isis enable OTUS
!
ip routing
!
router isis OTUS
   net 49.0011.0100.0100.0002.00
   is-type level-2
   !
   address-family ipv4 unicast
      bfd all-interfaces
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
   isis enable OTUS
   isis network point-to-point
   isis authentication mode text
   isis authentication key 7 Y5rbfDnzn74=
!
interface Ethernet2
   description ### to_Spine-2_eth1 ###
   no switchport
   ip address 10.1.5.7/31
   isis enable OTUS
   isis network point-to-point
   isis authentication mode text
   isis authentication key 7 Y5rbfDnzn74=
!
interface Loopback0
   ip address 10.1.1.1/32
   isis enable OTUS
!
ip routing
!
router isis OTUS
   net 49.0011.0100.0100.1001.00
   is-type level-2
   !
   address-family ipv4 unicast
      bfd all-interfaces
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
   isis enable OTUS
   isis network point-to-point
   isis authentication mode text
   isis authentication key 7 Y5rbfDnzn74=
!
interface Ethernet2
   description ### to_Spine-2_eth2 ###
   no switchport
   ip address 10.1.5.9/31
   isis enable OTUS
   isis network point-to-point
   isis authentication mode text
   isis authentication key 7 Y5rbfDnzn74=
!
interface Loopback0
   ip address 10.1.1.2/32
   isis enable OTUS
!
ip routing
!
router isis OTUS
   net 49.0011.0100.0100.1002.00
   is-type level-2
   !
   address-family ipv4 unicast
      bfd all-interfaces
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
   isis enable OTUS
   isis network point-to-point
   isis authentication mode text
   isis authentication key 7 Y5rbfDnzn74=
!
interface Ethernet2
   description ### to_Spine-2_eth3 ###
   no switchport
   ip address 10.1.5.11/31
   isis enable OTUS
   isis network point-to-point
   isis authentication mode text
   isis authentication key 7 Y5rbfDnzn74=
!
interface Loopback0
   ip address 10.1.1.3/32
   isis enable OTUS
!
ip routing
!
router isis OTUS
   net 49.0011.0100.0100.1003.00
   is-type level-2
   !
   address-family ipv4 unicast
      bfd all-interfaces
!
```

#### 4. Демонстрация работы настроек протокола OSPF

#### Spine-1
```
```

```
```

```
```

##### Spine-2
```
```

```
```

```
```

##### Leaf-1
```
```

```
```

```
```

##### Leaf-2 
```
```

```
```

```
```

##### Leaf-3
```
```

```
```

```
```

