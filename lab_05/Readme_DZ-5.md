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

```
##### Leaf-01
```

```
##### Leaf-02 
```

```
##### Leaf-03
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