# Домашнее задание №4.
## Underlay. eBGP.
### Цель:
Настроить eBGP для Underlay сети

### Решение.
#### 1. Схема сети

Для Underlay сети реализуем eBGP связность, выделив SPINE в одну AS и каждый LEAF в уникальную AS. 

![dz-4_topo_ebgp_underlay.png](dz-4_topo_ebgp_underlay.png)

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
   no ip ospf bfd
!
interface Ethernet2
   description ### to_Leaf-2_eth1 ###
   no switchport
   ip address 10.1.5.2/31
   no ip ospf bfd
!
interface Ethernet3
   description ### to_Leaf-3_eth1 ###
   no switchport
   ip address 10.1.5.4/31
   no ip ospf bfd
!
interface Loopback0
   ip address 10.1.0.1/32
!
ip routing
!
peer-filter LAB_OTUS_COD
   10 match as-range 65001-65003 result accept
!
router bgp 65000
   router-id 10.1.0.1
   maximum-paths 3
   bgp listen range 10.1.0.0/16 peer-group OTUS_LEAF peer-filter LAB_OTUS_COD
   neighbor OTUS_LEAF peer-group
   neighbor OTUS_LEAF password 7 +/ddbal+Y1I=
   neighbor OTUS_LEAF maximum-routes 12000
   network 10.1.0.1/32
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
   no ip ospf bfd
!
interface Ethernet2
   description ### to_Leaf-2_eth2 ###
   no switchport
   ip address 10.1.5.8/31
   no ip ospf bfd
!
interface Ethernet3
   description ### to_Leaf-3_eth2 ###
   no switchport
   ip address 10.1.5.10/31
   no ip ospf bfd
!
interface Loopback0
   ip address 10.1.0.2/32
!
ip routing
!
peer-filter LAB_OTUS_COD
   10 match as-range 65001-65003 result accept
!
router bgp 65000
   router-id 10.1.0.2
   maximum-paths 3
   bgp listen range 10.1.0.0/16 peer-group OTUS_LEAF peer-filter LAB_OTUS_COD
   neighbor OTUS_LEAF peer-group
   neighbor OTUS_LEAF password 7 +/ddbal+Y1I=
   neighbor OTUS_LEAF maximum-routes 12000
   network 10.1.0.2/32
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
   no ip ospf bfd
!
interface Ethernet2
   description ### to_Spine-2_eth1 ###
   no switchport
   ip address 10.1.5.7/31
   no ip ospf bfd
!
interface Loopback0
   ip address 10.1.1.1/32
!
ip routing
!
router bgp 65001
   router-id 10.1.1.1
   maximum-paths 3
   neighbor OTUS_SPINE peer-group
   neighbor OTUS_SPINE remote-as 65000
   neighbor OTUS_SPINE fall-over bfd
   neighbor OTUS_SPINE password 7 kCt/R/lTQ8E=
   neighbor OTUS_SPINE maximum-routes 12000
   neighbor 10.1.5.0 peer-group OTUS_SPINE
   neighbor 10.1.5.6 peer-group OTUS_SPINE
   network 10.1.1.1/32
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
   no ip ospf bfd
!
interface Ethernet2
   description ### to_Spine-2_eth2 ###
   no switchport
   ip address 10.1.5.9/31
   no ip ospf bfd
!
interface Loopback0
   ip address 10.1.1.2/32
!
ip routing
!
router bgp 65002
   router-id 10.1.1.2
   maximum-paths 3
   neighbor OTUS_SPINE peer-group
   neighbor OTUS_SPINE remote-as 65000
   neighbor OTUS_SPINE fall-over bfd
   neighbor OTUS_SPINE password 7 kCt/R/lTQ8E=
   neighbor OTUS_SPINE maximum-routes 12000
   neighbor 10.1.5.2 peer-group OTUS_SPINE
   neighbor 10.1.5.8 peer-group OTUS_SPINE
   network 10.1.1.2/32
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
   no ip ospf bfd
!
interface Ethernet2
   description ### to_Spine-2_eth3 ###
   no switchport
   ip address 10.1.5.11/31
   no ip ospf bfd
!
interface Loopback0
   ip address 10.1.1.3/32
!
ip routing
!
router bgp 65003
   router-id 10.1.1.3
   maximum-paths 3
   neighbor OTUS_SPINE peer-group
   neighbor OTUS_SPINE remote-as 65000
   neighbor OTUS_SPINE fall-over bfd
   neighbor OTUS_SPINE password 7 kCt/R/lTQ8E=
   neighbor OTUS_SPINE maximum-routes 12000
   neighbor 10.1.5.4 peer-group OTUS_SPINE
   neighbor 10.1.5.10 peer-group OTUS_SPINE
   network 10.1.1.3/32
!
```

#### 4. Демонстрация работы настроек протокола eBGP.

#### Spine-1
```
```

```
```

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

```
```

```
```
