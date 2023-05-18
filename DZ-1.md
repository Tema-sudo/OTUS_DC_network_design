# Домашнее задание №1.
## Проектирование адресного пространства.
### Цель:
- Собрать схему CLOS;
- Распределить адресное пространство.

### Решение.
#### 1. Топология сети CLOS ####

![dz-1_topo_clos](lab_01/dz-1_topo_clos.png)

#### 2. Распределение адресного пространства ####
Для распределения адресного пространства в ЦОД выбран частный диапазон IP сетей: 10.0.0.0/8.

Формат IP-адреса: 10.X.Y.Z, где
- X - номер дата центра;
- Y - распределяется как:
Y=0 - Loopback Spine; 
Y=1 - Loopback Leaf;
Y=2-4 - reserved; 
Y=5-9 - p2p link;
Y=255, (255-1.... )- services;
- Z - значение по порядку.

Таблица адресов:

|Name|Loopback0|INT-0/1|INT-0/2|INT-0/3|
|---|---|---|---|---|
Spine-1|10.1.0.1/32|10.1.5.1/30|10.1.5.5/30|10.1.5.9/30|
Spine-2|10.1.0.2/32|10.1.5.13/30|10.1.5.17/30|10.1.5.21/30|
Leaf-1|10.1.1.1/32|10.1.5.2/30|10.1.5.14/30|N/A|
Leaf-2|10.1.1.2/32|10.1.5.6/30|10.1.5.18/30|N/A|
Leaf-3|10.1.1.3/32|10.1.5.10/30|10.1.5.22/30|N/A|

#### 3. Настройки ####

Лабораторный стенд развернут на ПО:
```
Arista vEOS
System MAC address:  5000.00d7.ee0b
Software image version: 4.20.1F
Architecture:           i386
Internal build version: 4.20.1F-6820520.4201F
Internal build ID:      790a11e8-5aaf-4be7-a11a-e61795d05b91
```
##### Spine-1 #####
```
!
hostname Spine-1
!
interface Ethernet1
   description ### to_Leaf-1_eth1 ###
   no switchport
   ip address 10.1.5.1/30
!
interface Ethernet2
   description ### to_Leaf-2_eth1 ###
   no switchport
   ip address 10.1.5.5/30
!
interface Ethernet3
   description ### to_Leaf-3_eth1 ###
   no switchport
   ip address 10.1.5.9/30
!
interface Loopback0
   ip address 10.1.0.1/32
!
```

##### Spine-2 #####

```
!
hostname Spine-2
!
interface Ethernet1
   description ### to_Leaf-1_eth2 ###
   no switchport
   ip address 10.1.5.13/30
!
interface Ethernet2
   description ### to_Leaf-2_eth2 ###
   no switchport
   ip address 10.1.5.17/30
!
interface Ethernet3
   description ### to_Leaf-3_eth2 ###
   no switchport
   ip address 10.1.5.21/30
!
interface Loopback0
   ip address 10.1.0.2/32
!
```

##### Leaf-1 #####

```
!
hostname Leaf-1
!
interface Ethernet1
   description ### to_Spine-1_eth1 ###
   no switchport
   ip address 10.1.5.2/30
!
interface Ethernet2
   description ### to_Spine-2_eth1 ###
   no switchport
   ip address 10.1.5.14/30
!
interface Loopback0
   ip address 10.1.1.1/32
!
```

##### Leaf-2 #####

```
!
hostname Leaf-2
!
interface Ethernet1
   description ### to_Spine-1_eth2 ###
   no switchport
   ip address 10.1.5.6/30
!
interface Ethernet2
   description ### to_Spine-2_eth2 ###
   no switchport
   ip address 10.1.5.18/30
!
interface Loopback0
   ip address 10.1.1.2/32
!
```

##### Leaf-3 #####

```
!
hostname Leaf-3
!
interface Ethernet1
   description ### to_Spine-1_eth3 ###
   no switchport
   ip address 10.1.5.10/30
!
interface Ethernet2
   description ### to_Spine-2_eth3 ###
   no switchport
   ip address 10.1.5.22/30
!
interface Loopback0
   ip address 10.1.1.3/32
!
```