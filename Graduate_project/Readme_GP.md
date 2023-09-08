## Проектная работа.
#### Проектирование геораспределенной сети ЦОД.
#### Автор: 
- Шатунов Роман;
- Коновалов Артём. 

#### Цель:
Cпроектировать и реализовать геораспределенную, катастрофоустройчивую фабрику ЦОД для надежной работы сервисов предприятия. 

#### Задачи:
1. Выработать унифицированное решение для возможности применения в энтерпрайз сетях средного размера;
2. Обеспечить гибкость использующихся протоколов и элементов топологии.     

#### Топология 
![GP_common_topo](GP_common_topo.png)

#### Решение
Собрали стенд в реалиях EVE-NG и выполнили планирование адресного простраства 
![GP_common_top_EVE-NG](GP_common_top_EVE-NG.jpg)

Выполнили настройки.

#### Проверка результата

Проверка IP связности в VRF-1

```
Srv-11#ping 10.128.0.2
PING 10.128.0.2 (10.128.0.2) 72(100) bytes of data.
80 bytes from 10.128.0.2: icmp_seq=1 ttl=64 time=181 ms
80 bytes from 10.128.0.2: icmp_seq=1 ttl=64 time=186 ms (DUP!)
80 bytes from 10.128.0.2: icmp_seq=2 ttl=64 time=203 ms
80 bytes from 10.128.0.2: icmp_seq=2 ttl=64 time=208 ms (DUP!)
80 bytes from 10.128.0.2: icmp_seq=3 ttl=64 time=235 ms
80 bytes from 10.128.0.2: icmp_seq=4 ttl=64 time=280 ms
80 bytes from 10.128.0.2: icmp_seq=3 ttl=64 time=298 ms (DUP!)
80 bytes from 10.128.0.2: icmp_seq=5 ttl=64 time=285 ms

--- 10.128.0.2 ping statistics ---
5 packets transmitted, 5 received, +3 duplicates, 0% packet loss, time 43ms
rtt min/avg/max/mdev = 181.073/235.025/298.580/44.216 ms, pipe 5, ipg/ewma 10.84      
```

```
Srv-21#ping 10.128.0.1
PING 10.128.0.1 (10.128.0.1) 72(100) bytes of data.
80 bytes from 10.128.0.1: icmp_seq=1 ttl=64 time=32.0 ms
80 bytes from 10.128.0.1: icmp_seq=2 ttl=64 time=24.0 ms
80 bytes from 10.128.0.1: icmp_seq=3 ttl=64 time=23.4 ms
80 bytes from 10.128.0.1: icmp_seq=4 ttl=64 time=25.2 ms
80 bytes from 10.128.0.1: icmp_seq=5 ttl=64 time=28.7 ms

--- 10.128.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 83ms
rtt min/avg/max/mdev = 23.421/26.705/32.048/3.251 ms, pipe 3, ipg/ewma 20.942/29.398 ms
```

Проверка IP связности в VRF-2

```
Srv-12#ping 10.129.0.2
PING 10.129.0.2 (10.129.0.2) 72(100) bytes of data.
80 bytes from 10.129.0.2: icmp_seq=1 ttl=64 time=28.9 ms
80 bytes from 10.129.0.2: icmp_seq=2 ttl=64 time=22.7 ms
80 bytes from 10.129.0.2: icmp_seq=3 ttl=64 time=21.8 ms
80 bytes from 10.129.0.2: icmp_seq=4 ttl=64 time=21.5 ms

--- 10.129.0.2 ping statistics ---
5 packets transmitted, 4 received, 20% packet loss, time 77ms
rtt min/avg/max/mdev = 21.572/23.778/28.945/3.018 ms, pipe 3, ipg/ewma 19.297/26.653 ms
```

```
Srv-22#ping 10.129.0.1
PING 10.129.0.1 (10.129.0.1) 72(100) bytes of data.
80 bytes from 10.129.0.1: icmp_seq=1 ttl=64 time=28.9 ms
80 bytes from 10.129.0.1: icmp_seq=2 ttl=64 time=24.3 ms
80 bytes from 10.129.0.1: icmp_seq=3 ttl=64 time=24.3 ms
80 bytes from 10.129.0.1: icmp_seq=4 ttl=64 time=24.1 ms
80 bytes from 10.129.0.1: icmp_seq=5 ttl=64 time=21.2 ms

--- 10.129.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 78ms
rtt min/avg/max/mdev = 21.296/24.615/28.954/2.460 ms, pipe 3, ipg/ewma 19.640/26.643 ms
```

Проверка IP связности устройств между VRF-1 и VRF-2

```
Srv-11#ping 10.129.0.1
PING 10.129.0.1 (10.129.0.1) 72(100) bytes of data.
80 bytes from 10.129.0.1: icmp_seq=1 ttl=59 time=65.9 ms
80 bytes from 10.129.0.1: icmp_seq=2 ttl=59 time=60.2 ms
80 bytes from 10.129.0.1: icmp_seq=3 ttl=59 time=52.7 ms
80 bytes from 10.129.0.1: icmp_seq=4 ttl=59 time=45.2 ms
80 bytes from 10.129.0.1: icmp_seq=5 ttl=59 time=49.1 ms

--- 10.129.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 44ms
rtt min/avg/max/mdev = 45.222/54.658/65.967/7.516 ms, pipe 5, ipg/ewma 11.172/59.848 ms

Srv-11#ping 10.129.0.2
PING 10.129.0.2 (10.129.0.2) 72(100) bytes of data.
80 bytes from 10.129.0.2: icmp_seq=1 ttl=59 time=61.4 ms
80 bytes from 10.129.0.2: icmp_seq=2 ttl=59 time=53.4 ms
80 bytes from 10.129.0.2: icmp_seq=3 ttl=59 time=48.1 ms
80 bytes from 10.129.0.2: icmp_seq=4 ttl=59 time=60.3 ms
80 bytes from 10.129.0.2: icmp_seq=5 ttl=59 time=53.7 ms

--- 10.129.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 44ms
rtt min/avg/max/mdev = 48.195/55.457/61.462/4.894 ms, pipe 5, ipg/ewma 11.001/58.443 ms
Srv-11#     
```

```
Srv-12#ping 10.128.0.1
PING 10.128.0.1 (10.128.0.1) 72(100) bytes of data.
80 bytes from 10.128.0.1: icmp_seq=1 ttl=59 time=58.2 ms
80 bytes from 10.128.0.1: icmp_seq=2 ttl=59 time=50.8 ms
80 bytes from 10.128.0.1: icmp_seq=3 ttl=59 time=46.9 ms
80 bytes from 10.128.0.1: icmp_seq=4 ttl=59 time=44.5 ms
80 bytes from 10.128.0.1: icmp_seq=5 ttl=59 time=42.5 ms

--- 10.128.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 43ms
rtt min/avg/max/mdev = 42.575/48.655/58.291/5.555 ms, pipe 5, ipg/ewma 10.977/53.121 ms

Srv-12#ping 10.128.0.2
PING 10.128.0.2 (10.128.0.2) 72(100) bytes of data.
From 10.3.1.1 icmp_seq=1 Time to live exceeded
From 10.3.1.1 icmp_seq=2 Time to live exceeded
From 10.3.1.1 icmp_seq=3 Time to live exceeded
From 10.3.1.1 icmp_seq=4 Time to live exceeded
From 10.3.1.1 icmp_seq=5 Time to live exceeded

--- 10.128.0.2 ping statistics ---
5 packets transmitted, 0 received, +5 errors, 100% packet loss, time 44ms
pipe 5
Srv-12#
```

```
Srv-21#ping 10.129.0.1
PING 10.129.0.1 (10.129.0.1) 72(100) bytes of data.
80 bytes from 10.129.0.1: icmp_seq=1 ttl=47 time=622 ms
80 bytes from 10.129.0.1: icmp_seq=3 ttl=59 time=710 ms
80 bytes from 10.129.0.1: icmp_seq=4 ttl=59 time=913 ms
80 bytes from 10.129.0.1: icmp_seq=5 ttl=59 time=907 ms

--- 10.129.0.1 ping statistics ---
5 packets transmitted, 4 received, 20% packet loss, time 42ms
rtt min/avg/max/mdev = 622.962/788.655/913.548/125.935 ms, pipe 5, ipg/ewma 10.623/698.713 ms

Srv-21#ping 10.129.0.2
PING 10.129.0.2 (10.129.0.2) 72(100) bytes of data.
80 bytes from 10.129.0.2: icmp_seq=1 ttl=59 time=56.6 ms
80 bytes from 10.129.0.2: icmp_seq=2 ttl=59 time=51.5 ms
80 bytes from 10.129.0.2: icmp_seq=3 ttl=59 time=51.9 ms
80 bytes from 10.129.0.2: icmp_seq=4 ttl=59 time=44.8 ms
80 bytes from 10.129.0.2: icmp_seq=5 ttl=59 time=37.9 ms

--- 10.129.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 44ms
rtt min/avg/max/mdev = 37.973/48.601/56.604/6.498 ms, pipe 5, ipg/ewma 11.202/52.128 ms
Srv-21#
```

```
Srv-22#ping 10.128.0.1
PING 10.128.0.1 (10.128.0.1) 72(100) bytes of data.
80 bytes from 10.128.0.1: icmp_seq=1 ttl=59 time=43.4 ms
80 bytes from 10.128.0.1: icmp_seq=2 ttl=59 time=37.4 ms
80 bytes from 10.128.0.1: icmp_seq=3 ttl=59 time=31.9 ms
80 bytes from 10.128.0.1: icmp_seq=4 ttl=59 time=35.2 ms
80 bytes from 10.128.0.1: icmp_seq=5 ttl=59 time=36.3 ms

--- 10.128.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 75ms
rtt min/avg/max/mdev = 31.986/36.896/43.436/3.755 ms, pipe 4, ipg/ewma 18.753/40.060 ms

Srv-22#ping 10.128.0.2
PING 10.128.0.2 (10.128.0.2) 72(100) bytes of data.
80 bytes from 10.128.0.2: icmp_seq=1 ttl=59 time=48.4 ms
80 bytes from 10.128.0.2: icmp_seq=2 ttl=59 time=44.0 ms
80 bytes from 10.128.0.2: icmp_seq=3 ttl=59 time=43.4 ms
80 bytes from 10.128.0.2: icmp_seq=4 ttl=59 time=42.0 ms
80 bytes from 10.128.0.2: icmp_seq=5 ttl=59 time=39.0 ms

--- 10.128.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 42ms
rtt min/avg/max/mdev = 39.054/43.425/48.431/3.060 ms, pipe 5, ipg/ewma 10.614/45.726 ms
Srv-22#
```

Проверка IP связности с "Интернет"

```
Srv-11#ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 72(100) bytes of data.
80 bytes from 8.8.8.8: icmp_seq=1 ttl=62 time=22.4 ms
80 bytes from 8.8.8.8: icmp_seq=2 ttl=62 time=63.6 ms
80 bytes from 8.8.8.8: icmp_seq=3 ttl=62 time=43.2 ms
80 bytes from 8.8.8.8: icmp_seq=4 ttl=62 time=24.7 ms
80 bytes from 8.8.8.8: icmp_seq=5 ttl=62 time=15.7 ms

--- 8.8.8.8 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 87ms
rtt min/avg/max/mdev = 15.793/33.982/63.691/17.428 ms, pipe 3, ipg/ewma 21.918/27.304 ms
Srv-11#
```

```
Srv-21#ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 72(100) bytes of data.
80 bytes from 8.8.8.8: icmp_seq=1 ttl=62 time=20.2 ms
80 bytes from 8.8.8.8: icmp_seq=2 ttl=62 time=16.6 ms
80 bytes from 8.8.8.8: icmp_seq=3 ttl=62 time=16.7 ms
80 bytes from 8.8.8.8: icmp_seq=4 ttl=62 time=16.8 ms
80 bytes from 8.8.8.8: icmp_seq=5 ttl=62 time=17.8 ms

--- 8.8.8.8 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 70ms
rtt min/avg/max/mdev = 16.668/17.656/20.210/1.347 ms, pipe 2, ipg/ewma 17.634/18.914 ms
Srv-21#
```

```
Srv-12#ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 72(100) bytes of data.
80 bytes from 8.8.8.8: icmp_seq=1 ttl=62 time=21.2 ms
80 bytes from 8.8.8.8: icmp_seq=2 ttl=62 time=52.9 ms
80 bytes from 8.8.8.8: icmp_seq=3 ttl=62 time=39.2 ms
80 bytes from 8.8.8.8: icmp_seq=4 ttl=62 time=21.8 ms
80 bytes from 8.8.8.8: icmp_seq=5 ttl=62 time=14.9 ms

--- 8.8.8.8 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 83ms
rtt min/avg/max/mdev = 14.926/30.050/52.983/14.025 ms, pipe 3, ipg/ewma 20.804/24.899 ms
Srv-12#
```

```
Srv-22#ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 72(100) bytes of data.
80 bytes from 8.8.8.8: icmp_seq=1 ttl=62 time=27.6 ms
80 bytes from 8.8.8.8: icmp_seq=2 ttl=62 time=21.0 ms
80 bytes from 8.8.8.8: icmp_seq=3 ttl=62 time=18.2 ms
80 bytes from 8.8.8.8: icmp_seq=4 ttl=62 time=17.4 ms
80 bytes from 8.8.8.8: icmp_seq=5 ttl=62 time=17.0 ms

--- 8.8.8.8 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 73ms
rtt min/avg/max/mdev = 17.085/20.292/27.672/3.944 ms, pipe 3, ipg/ewma 18.389/23.770 ms
Srv-22#
```

Проверка наличия BGP-EVP маршрутов на Leaf устройствах

```
Leaf-11#show bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.1.0.1, local AS number 65101
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >     RD: 10.1.0.1:10 imet 10.1.1.1
                                 -                     -       -       0       i
 * >     RD: 10.1.0.1:20 imet 10.1.1.1
                                 -                     -       -       0       i
 * >Ec   RD: 10.9.0.1:10 imet 10.9.1.1
                                 10.9.1.1              -       100     0       65100 65199 65200 65201 i
 *  ec   RD: 10.9.0.1:10 imet 10.9.1.1
                                 10.9.1.1              -       100     0       65100 65199 65200 65201 i
 * >Ec   RD: 10.9.0.1:20 imet 10.9.1.1
                                 10.9.1.1              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.1:20 imet 10.9.1.1
                                 10.9.1.1              -       100     0       65100 65199 65200 65201 i
 * >Ec   RD: 10.9.0.2:10 imet 10.9.1.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.2:10 imet 10.9.1.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 * >Ec   RD: 10.9.0.2:20 imet 10.9.1.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.2:20 imet 10.9.1.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i

Leaf-11#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.1, local AS number 65101
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >     RD: 10.1.0.1:20 mac-ip 5000.0016.00e6
                                 -                     -       -       0       i
 * >     RD: 10.1.0.1:20 mac-ip 5000.0016.00e6 10.129.0.1
                                 -                     -       -       0       i
 * >Ec   RD: 10.9.0.1:20 mac-ip 5000.0074.4eca
                                 10.9.1.1              -       100     0       65100 65199 65200 65201 i
 *  ec   RD: 10.9.0.1:20 mac-ip 5000.0074.4eca
                                 10.9.1.1              -       100     0       65100 65199 65200 65201 i
 * >Ec   RD: 10.9.0.2:20 mac-ip 5000.0074.4eca
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.2:20 mac-ip 5000.0074.4eca
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 * >Ec   RD: 10.9.0.1:20 mac-ip 5000.0074.4eca 10.129.0.2
                                 10.9.1.1              -       100     0       65100 65199 65200 65201 i
 *  ec   RD: 10.9.0.1:20 mac-ip 5000.0074.4eca 10.129.0.2
                                 10.9.1.1              -       100     0       65100 65199 65200 65201 i
 * >Ec   RD: 10.9.0.2:20 mac-ip 5000.0074.4eca 10.129.0.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.2:20 mac-ip 5000.0074.4eca 10.129.0.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 * >     RD: 10.1.0.1:10 mac-ip 5000.00dd.cb9a
                                 -                     -       -       0       i
 * >     RD: 10.1.0.1:10 mac-ip 5000.00dd.cb9a 10.128.0.1
                                 -                     -       -       0       i
 * >Ec   RD: 10.9.0.1:10 mac-ip 5000.00f4.253d
                                 10.9.1.1              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.1:10 mac-ip 5000.00f4.253d
                                 10.9.1.1              -       100     0       65100 65299 65200 65201 i
 * >Ec   RD: 10.9.0.2:10 mac-ip 5000.00f4.253d
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.2:10 mac-ip 5000.00f4.253d
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 * >Ec   RD: 10.9.0.1:10 mac-ip 5000.00f4.253d 10.128.0.2
                                 10.9.1.1              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.1:10 mac-ip 5000.00f4.253d 10.128.0.2
                                 10.9.1.1              -       100     0       65100 65299 65200 65201 i
 * >Ec   RD: 10.9.0.2:10 mac-ip 5000.00f4.253d 10.128.0.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.2:10 mac-ip 5000.00f4.253d 10.128.0.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i

Leaf-11#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.1.0.1, local AS number 65101
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 10.1.0.3:10001 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65100 65199 65000 ?
 *  ec   RD: 10.1.0.3:10001 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65100 65199 65000 ?
 * >Ec   RD: 10.1.0.3:10002 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65100 65199 65000 ?
 *  ec   RD: 10.1.0.3:10002 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65100 65199 65000 ?
 * >Ec   RD: 10.1.0.3:10001 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65100 65199 i
 *  ec   RD: 10.1.0.3:10001 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65100 65199 i
 * >Ec   RD: 10.1.0.3:10002 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65100 65199 65000 65000 i
 *  ec   RD: 10.1.0.3:10002 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65100 65199 65000 65000 i
 * >Ec   RD: 10.1.0.3:10001 ip-prefix 10.129.0.0/16
                                 10.9.1.254            -       100     0       65100 65299 65000 65000 i
 *  ec   RD: 10.1.0.3:10001 ip-prefix 10.129.0.0/16
                                 10.9.1.254            -       100     0       65100 65299 65000 65000 i
 * >Ec   RD: 10.1.0.3:10002 ip-prefix 10.129.0.0/16
                                 10.1.1.3              -       100     0       65100 65199 i
 *  ec   RD: 10.1.0.3:10002 ip-prefix 10.129.0.0/16
                                 10.1.1.3              -       100     0       65100 65199 i
Leaf-11#
```

```
Leaf-12#show bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.1.0.2, local AS number 65101
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >     RD: 10.1.0.2:10 imet 10.1.1.2
                                 -                     -       -       0       i
 * >     RD: 10.1.0.2:20 imet 10.1.1.2
                                 -                     -       -       0       i
 * >Ec   RD: 10.9.0.1:10 imet 10.9.1.1
                                 10.9.1.1              -       100     0       65100 65199 65200 65201 i
 *  ec   RD: 10.9.0.1:10 imet 10.9.1.1
                                 10.9.1.1              -       100     0       65100 65199 65200 65201 i
 * >Ec   RD: 10.9.0.1:20 imet 10.9.1.1
                                 10.9.1.1              -       100     0       65100 65199 65200 65201 i
 *  ec   RD: 10.9.0.1:20 imet 10.9.1.1
                                 10.9.1.1              -       100     0       65100 65299 65200 65201 i
 * >Ec   RD: 10.9.0.2:10 imet 10.9.1.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.2:10 imet 10.9.1.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 * >Ec   RD: 10.9.0.2:20 imet 10.9.1.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.2:20 imet 10.9.1.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i

Leaf-12#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.1.0.2, local AS number 65101
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >     RD: 10.1.0.2:20 mac-ip 5000.0016.00e6
                                 -                     -       -       0       i
 * >     RD: 10.1.0.2:20 mac-ip 5000.0016.00e6 10.129.0.1
                                 -                     -       -       0       i
 * >Ec   RD: 10.9.0.1:20 mac-ip 5000.0074.4eca
                                 10.9.1.1              -       100     0       65100 65199 65200 65201 i
 *  ec   RD: 10.9.0.1:20 mac-ip 5000.0074.4eca
                                 10.9.1.1              -       100     0       65100 65199 65200 65201 i
 * >Ec   RD: 10.9.0.2:20 mac-ip 5000.0074.4eca
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.2:20 mac-ip 5000.0074.4eca
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 * >Ec   RD: 10.9.0.1:20 mac-ip 5000.0074.4eca 10.129.0.2
                                 10.9.1.1              -       100     0       65100 65199 65200 65201 i
 *  ec   RD: 10.9.0.1:20 mac-ip 5000.0074.4eca 10.129.0.2
                                 10.9.1.1              -       100     0       65100 65199 65200 65201 i
 * >Ec   RD: 10.9.0.2:20 mac-ip 5000.0074.4eca 10.129.0.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.2:20 mac-ip 5000.0074.4eca 10.129.0.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 * >     RD: 10.1.0.2:10 mac-ip 5000.00dd.cb9a
                                 -                     -       -       0       i
 * >     RD: 10.1.0.2:10 mac-ip 5000.00dd.cb9a 10.128.0.1
                                 -                     -       -       0       i
 * >Ec   RD: 10.9.0.1:10 mac-ip 5000.00f4.253d
                                 10.9.1.1              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.1:10 mac-ip 5000.00f4.253d
                                 10.9.1.1              -       100     0       65100 65299 65200 65201 i
 * >Ec   RD: 10.9.0.2:10 mac-ip 5000.00f4.253d
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.2:10 mac-ip 5000.00f4.253d
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 * >Ec   RD: 10.9.0.1:10 mac-ip 5000.00f4.253d 10.128.0.2
                                 10.9.1.1              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.1:10 mac-ip 5000.00f4.253d 10.128.0.2
                                 10.9.1.1              -       100     0       65100 65299 65200 65201 i
 * >Ec   RD: 10.9.0.2:10 mac-ip 5000.00f4.253d 10.128.0.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i
 *  ec   RD: 10.9.0.2:10 mac-ip 5000.00f4.253d 10.128.0.2
                                 10.9.1.2              -       100     0       65100 65299 65200 65201 i

Leaf-12#show bgp evpn route-type ip-prefix
% Incomplete command
Leaf-12#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.1.0.2, local AS number 65101
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 10.1.0.3:10001 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65100 65199 65000 ?
 *  ec   RD: 10.1.0.3:10001 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65100 65199 65000 ?
 * >Ec   RD: 10.1.0.3:10002 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65100 65199 65000 ?
 *  ec   RD: 10.1.0.3:10002 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65100 65199 65000 ?
 * >Ec   RD: 10.1.0.3:10001 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65100 65199 i
 *  ec   RD: 10.1.0.3:10001 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65100 65199 i
 * >Ec   RD: 10.1.0.3:10002 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65100 65199 65000 65000 i
 *  ec   RD: 10.1.0.3:10002 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65100 65199 65000 65000 i
 * >Ec   RD: 10.1.0.3:10001 ip-prefix 10.129.0.0/16
                                 10.9.1.254            -       100     0       65100 65299 65000 65000 i
 *  ec   RD: 10.1.0.3:10001 ip-prefix 10.129.0.0/16
                                 10.9.1.254            -       100     0       65100 65299 65000 65000 i
 * >Ec   RD: 10.1.0.3:10002 ip-prefix 10.129.0.0/16
                                 10.1.1.3              -       100     0       65100 65199 i
 *  ec   RD: 10.1.0.3:10002 ip-prefix 10.129.0.0/16
                                 10.1.1.3              -       100     0       65100 65199 i
Leaf-12#
```

```
Leaf-21#show bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.9.0.1, local AS number 65201
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 10.1.0.1:10 imet 10.1.1.1
                                 10.1.1.1              -       100     0       65200 65199 65100 65101 i
 *  ec   RD: 10.1.0.1:10 imet 10.1.1.1
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.1:20 imet 10.1.1.1
                                 10.1.1.1              -       100     0       65200 65199 65100 65101 i
 *  ec   RD: 10.1.0.1:20 imet 10.1.1.1
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.2:10 imet 10.1.1.2
                                 10.1.1.2              -       100     0       65200 65199 65100 65101 i
 *  ec   RD: 10.1.0.2:10 imet 10.1.1.2
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.2:20 imet 10.1.1.2
                                 10.1.1.2              -       100     0       65200 65199 65100 65101 i
 *  ec   RD: 10.1.0.2:20 imet 10.1.1.2
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 * >     RD: 10.9.0.1:10 imet 10.9.1.1
                                 -                     -       -       0       i
 * >     RD: 10.9.0.1:20 imet 10.9.1.1
                                 -                     -       -       0       i

Leaf-21#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.9.0.1, local AS number 65201
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 10.1.0.1:20 mac-ip 5000.0016.00e6
                                 10.1.1.1              -       100     0       65200 65199 65100 65101 i
 *  ec   RD: 10.1.0.1:20 mac-ip 5000.0016.00e6
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.2:20 mac-ip 5000.0016.00e6
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 *  ec   RD: 10.1.0.2:20 mac-ip 5000.0016.00e6
                                 10.1.1.2              -       100     0       65200 65199 65100 65101 i
 * >Ec   RD: 10.1.0.1:20 mac-ip 5000.0016.00e6 10.129.0.1
                                 10.1.1.1              -       100     0       65200 65199 65100 65101 i
 *  ec   RD: 10.1.0.1:20 mac-ip 5000.0016.00e6 10.129.0.1
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.2:20 mac-ip 5000.0016.00e6 10.129.0.1
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 *  ec   RD: 10.1.0.2:20 mac-ip 5000.0016.00e6 10.129.0.1
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 * >     RD: 10.9.0.1:20 mac-ip 5000.0074.4eca
                                 -                     -       -       0       i
 * >     RD: 10.9.0.1:20 mac-ip 5000.0074.4eca 10.129.0.2
                                 -                     -       -       0       i
 * >Ec   RD: 10.1.0.1:10 mac-ip 5000.00dd.cb9a
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 *  ec   RD: 10.1.0.1:10 mac-ip 5000.00dd.cb9a
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.2:10 mac-ip 5000.00dd.cb9a
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 *  ec   RD: 10.1.0.2:10 mac-ip 5000.00dd.cb9a
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.1:10 mac-ip 5000.00dd.cb9a 10.128.0.1
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 *  ec   RD: 10.1.0.1:10 mac-ip 5000.00dd.cb9a 10.128.0.1
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.2:10 mac-ip 5000.00dd.cb9a 10.128.0.1
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 *  ec   RD: 10.1.0.2:10 mac-ip 5000.00dd.cb9a 10.128.0.1
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 * >     RD: 10.9.0.1:10 mac-ip 5000.00f4.253d
                                 -                     -       -       0       i
 * >     RD: 10.9.0.1:10 mac-ip 5000.00f4.253d 10.128.0.2
                                 -                     -       -       0       i

Leaf-21#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.9.0.1, local AS number 65201
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 10.1.0.3:10001 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65200 65199 65000 ?
 *  ec   RD: 10.1.0.3:10001 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65200 65199 65000 ?
 * >Ec   RD: 10.1.0.3:10002 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65200 65199 65000 ?
 *  ec   RD: 10.1.0.3:10002 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65200 65199 65000 ?
 * >Ec   RD: 10.1.0.3:10001 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65200 65199 i
 *  ec   RD: 10.1.0.3:10001 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65200 65199 i
 * >Ec   RD: 10.1.0.3:10002 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65200 65199 65000 65000 i
 *  ec   RD: 10.1.0.3:10002 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65200 65199 65000 65000 i
 * >Ec   RD: 10.1.0.3:10001 ip-prefix 10.129.0.0/16
                                 10.9.1.254            -       100     0       65200 65299 65000 65000 i
 *  ec   RD: 10.1.0.3:10001 ip-prefix 10.129.0.0/16
                                 10.9.1.254            -       100     0       65200 65299 65000 65000 i
 * >Ec   RD: 10.1.0.3:10002 ip-prefix 10.129.0.0/16
                                 10.1.1.3              -       100     0       65200 65199 i
 *  ec   RD: 10.1.0.3:10002 ip-prefix 10.129.0.0/16
                                 10.1.1.3              -       100     0       65200 65199 i
Leaf-21#
```

```
Leaf-22#show bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.9.0.2, local AS number 65201
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 10.1.0.1:10 imet 10.1.1.1
                                 10.1.1.1              -       100     0       65200 65199 65100 65101 i
 *  ec   RD: 10.1.0.1:10 imet 10.1.1.1
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.1:20 imet 10.1.1.1
                                 10.1.1.1              -       100     0       65200 65199 65100 65101 i
 *  ec   RD: 10.1.0.1:20 imet 10.1.1.1
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.2:10 imet 10.1.1.2
                                 10.1.1.2              -       100     0       65200 65199 65100 65101 i
 *  ec   RD: 10.1.0.2:10 imet 10.1.1.2
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.2:20 imet 10.1.1.2
                                 10.1.1.2              -       100     0       65200 65199 65100 65101 i
 *  ec   RD: 10.1.0.2:20 imet 10.1.1.2
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 * >     RD: 10.9.0.2:10 imet 10.9.1.2
                                 -                     -       -       0       i
 * >     RD: 10.9.0.2:20 imet 10.9.1.2
                                 -                     -       -       0       i

Leaf-22#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.9.0.2, local AS number 65201
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 10.1.0.1:20 mac-ip 5000.0016.00e6
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 *  ec   RD: 10.1.0.1:20 mac-ip 5000.0016.00e6
                                 10.1.1.1              -       100     0       65200 65199 65100 65101 i
 * >Ec   RD: 10.1.0.2:20 mac-ip 5000.0016.00e6
                                 10.1.1.2              -       100     0       65200 65199 65100 65101 i
 *  ec   RD: 10.1.0.2:20 mac-ip 5000.0016.00e6
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.1:20 mac-ip 5000.0016.00e6 10.129.0.1
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 *  ec   RD: 10.1.0.1:20 mac-ip 5000.0016.00e6 10.129.0.1
                                 10.1.1.1              -       100     0       65200 65199 65100 65101 i
 * >Ec   RD: 10.1.0.2:20 mac-ip 5000.0016.00e6 10.129.0.1
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 *  ec   RD: 10.1.0.2:20 mac-ip 5000.0016.00e6 10.129.0.1
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 * >     RD: 10.9.0.2:20 mac-ip 5000.0074.4eca
                                 -                     -       -       0       i
 * >     RD: 10.9.0.2:20 mac-ip 5000.0074.4eca 10.129.0.2
                                 -                     -       -       0       i
 * >Ec   RD: 10.1.0.1:10 mac-ip 5000.00dd.cb9a
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 *  ec   RD: 10.1.0.1:10 mac-ip 5000.00dd.cb9a
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.2:10 mac-ip 5000.00dd.cb9a
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 *  ec   RD: 10.1.0.2:10 mac-ip 5000.00dd.cb9a
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.1:10 mac-ip 5000.00dd.cb9a 10.128.0.1
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 *  ec   RD: 10.1.0.1:10 mac-ip 5000.00dd.cb9a 10.128.0.1
                                 10.1.1.1              -       100     0       65200 65299 65100 65101 i
 * >Ec   RD: 10.1.0.2:10 mac-ip 5000.00dd.cb9a 10.128.0.1
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 *  ec   RD: 10.1.0.2:10 mac-ip 5000.00dd.cb9a 10.128.0.1
                                 10.1.1.2              -       100     0       65200 65299 65100 65101 i
 * >     RD: 10.9.0.2:10 mac-ip 5000.00f4.253d
                                 -                     -       -       0       i
 * >     RD: 10.9.0.2:10 mac-ip 5000.00f4.253d 10.128.0.2
                                 -                     -       -       0       i

Leaf-22#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.9.0.2, local AS number 65201
Route status codes: s - suppressed, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 10.1.0.3:10001 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65200 65199 65000 ?
 *  ec   RD: 10.1.0.3:10001 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65200 65199 65000 ?
 * >Ec   RD: 10.1.0.3:10002 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65200 65199 65000 ?
 *  ec   RD: 10.1.0.3:10002 ip-prefix 0.0.0.0/0
                                 10.1.1.3              -       100     0       65200 65199 65000 ?
 * >Ec   RD: 10.1.0.3:10001 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65200 65199 i
 *  ec   RD: 10.1.0.3:10001 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65200 65199 i
 * >Ec   RD: 10.1.0.3:10002 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65200 65199 65000 65000 i
 *  ec   RD: 10.1.0.3:10002 ip-prefix 10.128.0.0/16
                                 10.1.1.3              -       100     0       65200 65199 65000 65000 i
 * >Ec   RD: 10.1.0.3:10001 ip-prefix 10.129.0.0/16
                                 10.9.1.254            -       100     0       65200 65299 65000 65000 i
 *  ec   RD: 10.1.0.3:10001 ip-prefix 10.129.0.0/16
                                 10.9.1.254            -       100     0       65200 65299 65000 65000 i
 * >Ec   RD: 10.1.0.3:10002 ip-prefix 10.129.0.0/16
                                 10.1.1.3              -       100     0       65200 65199 i
 *  ec   RD: 10.1.0.3:10002 ip-prefix 10.129.0.0/16
                                 10.1.1.3              -       100     0       65200 65199 i
Leaf-22#
```

Проверка VRF таблицы маршрутизации на Leaf

```
Leaf-11#show ip route vrf VRF-1

VRF: VRF-1
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via VTEP 10.1.1.3 VNI 10001 router-mac 50:00:00:00:51:83 local-interface Vxlan1

 B E      10.128.0.2/32 [200/0] via VTEP 10.9.1.1 VNI 10001 router-mac 50:00:00:6b:0f:11 local-interface Vxlan1
                                via VTEP 10.9.1.2 VNI 10001 router-mac 50:00:00:5b:6f:f5 local-interface Vxlan1
 C        10.128.0.0/24 is directly connected, Vlan10
 B E      10.128.0.0/16 [200/0] via VTEP 10.1.1.3 VNI 10001 router-mac 50:00:00:00:51:83 local-interface Vxlan1
 B E      10.129.0.0/16 [200/0] via VTEP 10.9.1.254 VNI 10001 router-mac 50:00:00:6f:20:c3 local-interface Vxlan1

Leaf-11#show ip route vrf VRF-2

VRF: VRF-2
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via VTEP 10.1.1.3 VNI 10002 router-mac 50:00:00:00:51:83 local-interface Vxlan1

 B E      10.128.0.0/16 [200/0] via VTEP 10.1.1.3 VNI 10002 router-mac 50:00:00:00:51:83 local-interface Vxlan1
 B E      10.129.0.2/32 [200/0] via VTEP 10.9.1.1 VNI 10002 router-mac 50:00:00:6b:0f:11 local-interface Vxlan1
                                via VTEP 10.9.1.2 VNI 10002 router-mac 50:00:00:5b:6f:f5 local-interface Vxlan1
 C        10.129.0.0/24 is directly connected, Vlan20
 B E      10.129.0.0/16 [200/0] via VTEP 10.1.1.3 VNI 10002 router-mac 50:00:00:00:51:83 local-interface Vxlan1

Leaf-11#
```

```
Leaf-12#show ip route vrf VRF-1

VRF: VRF-1
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via VTEP 10.1.1.3 VNI 10001 router-mac 50:00:00:00:51:83 local-interface Vxlan1

 B E      10.128.0.2/32 [200/0] via VTEP 10.9.1.1 VNI 10001 router-mac 50:00:00:6b:0f:11 local-interface Vxlan1
                                via VTEP 10.9.1.2 VNI 10001 router-mac 50:00:00:5b:6f:f5 local-interface Vxlan1
 C        10.128.0.0/24 is directly connected, Vlan10
 B E      10.128.0.0/16 [200/0] via VTEP 10.1.1.3 VNI 10001 router-mac 50:00:00:00:51:83 local-interface Vxlan1
 B E      10.129.0.0/16 [200/0] via VTEP 10.9.1.254 VNI 10001 router-mac 50:00:00:6f:20:c3 local-interface Vxlan1


Leaf-12#show ip route vrf VRF-2

VRF: VRF-2
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via VTEP 10.1.1.3 VNI 10002 router-mac 50:00:00:00:51:83 local-interface Vxlan1

 B E      10.128.0.0/16 [200/0] via VTEP 10.1.1.3 VNI 10002 router-mac 50:00:00:00:51:83 local-interface Vxlan1
 B E      10.129.0.2/32 [200/0] via VTEP 10.9.1.1 VNI 10002 router-mac 50:00:00:6b:0f:11 local-interface Vxlan1
                                via VTEP 10.9.1.2 VNI 10002 router-mac 50:00:00:5b:6f:f5 local-interface Vxlan1
 C        10.129.0.0/24 is directly connected, Vlan20
 B E      10.129.0.0/16 [200/0] via VTEP 10.1.1.3 VNI 10002 router-mac 50:00:00:00:51:83 local-interface Vxlan1

Leaf-12#
```

```
Leaf-21#show ip route vrf VRF-1

VRF: VRF-1
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via VTEP 10.1.1.3 VNI 10001 router-mac 50:00:00:00:51:83 local-interface Vxlan1

 B E      10.128.0.1/32 [200/0] via VTEP 10.1.1.2 VNI 10001 router-mac 50:00:00:11:5d:47 local-interface Vxlan1
                                via VTEP 10.1.1.1 VNI 10001 router-mac 50:00:00:d0:a5:18 local-interface Vxlan1
 C        10.128.0.0/24 is directly connected, Vlan10
 B E      10.128.0.0/16 [200/0] via VTEP 10.1.1.3 VNI 10001 router-mac 50:00:00:00:51:83 local-interface Vxlan1
 B E      10.129.0.0/16 [200/0] via VTEP 10.9.1.254 VNI 10001 router-mac 50:00:00:6f:20:c3 local-interface Vxlan1

Leaf-21#
Leaf-21#
Leaf-21#show ip route vrf VRF-2

VRF: VRF-2
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via VTEP 10.1.1.3 VNI 10002 router-mac 50:00:00:00:51:83 local-interface Vxlan1

 B E      10.128.0.0/16 [200/0] via VTEP 10.1.1.3 VNI 10002 router-mac 50:00:00:00:51:83 local-interface Vxlan1
 B E      10.129.0.1/32 [200/0] via VTEP 10.1.1.2 VNI 10002 router-mac 50:00:00:11:5d:47 local-interface Vxlan1
                                via VTEP 10.1.1.1 VNI 10002 router-mac 50:00:00:d0:a5:18 local-interface Vxlan1
 C        10.129.0.0/24 is directly connected, Vlan20
 B E      10.129.0.0/16 [200/0] via VTEP 10.1.1.3 VNI 10002 router-mac 50:00:00:00:51:83 local-interface Vxlan1

Leaf-21#
```

```
Leaf-22#show ip route vrf VRF-1

VRF: VRF-1
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via VTEP 10.1.1.3 VNI 10001 router-mac 50:00:00:00:51:83 local-interface Vxlan1

 B E      10.128.0.1/32 [200/0] via VTEP 10.1.1.2 VNI 10001 router-mac 50:00:00:11:5d:47 local-interface Vxlan1
                                via VTEP 10.1.1.1 VNI 10001 router-mac 50:00:00:d0:a5:18 local-interface Vxlan1
 C        10.128.0.0/24 is directly connected, Vlan10
 B E      10.128.0.0/16 [200/0] via VTEP 10.1.1.3 VNI 10001 router-mac 50:00:00:00:51:83 local-interface Vxlan1
 B E      10.129.0.0/16 [200/0] via VTEP 10.9.1.254 VNI 10001 router-mac 50:00:00:6f:20:c3 local-interface Vxlan1

Leaf-22#show ip route vrf VRF-2

VRF: VRF-2
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via VTEP 10.1.1.3 VNI 10002 router-mac 50:00:00:00:51:83 local-interface Vxlan1

 B E      10.128.0.0/16 [200/0] via VTEP 10.1.1.3 VNI 10002 router-mac 50:00:00:00:51:83 local-interface Vxlan1
 B E      10.129.0.1/32 [200/0] via VTEP 10.1.1.2 VNI 10002 router-mac 50:00:00:11:5d:47 local-interface Vxlan1
                                via VTEP 10.1.1.1 VNI 10002 router-mac 50:00:00:d0:a5:18 local-interface Vxlan1
 C        10.129.0.0/24 is directly connected, Vlan20
 B E      10.129.0.0/16 [200/0] via VTEP 10.1.1.3 VNI 10002 router-mac 50:00:00:00:51:83 local-interface Vxlan1

Leaf-22#
```

Проверка VRF таблиц маршрутизации на BR-Leaf устройствах
```
BR-Leaf-11#show ip route vrf VRF-1

VRF: VRF-1
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via 10.3.1.0, Ethernet5.1

 C        10.3.1.0/31 is directly connected, Ethernet5.1
 C        10.11.1.0/31 is directly connected, Ethernet6.1
 B E      10.128.0.1/32 [200/0] via VTEP 10.1.1.2 VNI 10001 router-mac 50:00:00:11:5d:47 local-interface Vxlan1
                                via VTEP 10.1.1.1 VNI 10001 router-mac 50:00:00:d0:a5:18 local-interface Vxlan1
 B E      10.128.0.2/32 [200/0] via VTEP 10.9.1.1 VNI 10001 router-mac 50:00:00:6b:0f:11 local-interface Vxlan1
                                via VTEP 10.9.1.2 VNI 10001 router-mac 50:00:00:5b:6f:f5 local-interface Vxlan1
 B E      10.129.0.0/16 [200/0] via 10.3.1.0, Ethernet5.1

BR-Leaf-11#show ip route vrf VRF-2

VRF: VRF-2
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via 10.3.1.2, Ethernet5.2

 C        10.3.1.2/31 is directly connected, Ethernet5.2
 C        10.11.1.2/31 is directly connected, Ethernet6.2
 B E      10.128.0.0/16 [200/0] via 10.3.1.2, Ethernet5.2
 B E      10.129.0.1/32 [200/0] via VTEP 10.1.1.2 VNI 10002 router-mac 50:00:00:11:5d:47 local-interface Vxlan1
                                via VTEP 10.1.1.1 VNI 10002 router-mac 50:00:00:d0:a5:18 local-interface Vxlan1
 B E      10.129.0.2/32 [200/0] via VTEP 10.9.1.1 VNI 10002 router-mac 50:00:00:6b:0f:11 local-interface Vxlan1
                                via VTEP 10.9.1.2 VNI 10002 router-mac 50:00:00:5b:6f:f5 local-interface Vxlan1
```


```
BR-Leaf-21#show ip route vrf VRF-1

VRF: VRF-1
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via 10.11.2.0, Ethernet5.1

 C        10.3.2.0/31 is directly connected, Ethernet6.1
 C        10.11.2.0/31 is directly connected, Ethernet5.1
 B E      10.128.0.1/32 [200/0] via VTEP 10.1.1.1 VNI 10001 router-mac 50:00:00:d0:a5:18 local-interface Vxlan1
                                via VTEP 10.1.1.2 VNI 10001 router-mac 50:00:00:11:5d:47 local-interface Vxlan1
 B E      10.128.0.2/32 [200/0] via VTEP 10.9.1.1 VNI 10001 router-mac 50:00:00:6b:0f:11 local-interface Vxlan1
                                via VTEP 10.9.1.2 VNI 10001 router-mac 50:00:00:5b:6f:f5 local-interface Vxlan1
 B E      10.128.0.0/16 [200/0] via 10.11.2.0, Ethernet5.1
 B E      10.129.0.0/16 [200/0] via 10.11.2.0, Ethernet5.1

BR-Leaf-21#show ip route vrf VRF-2

VRF: VRF-2
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via 10.11.2.2, Ethernet5.2

 C        10.3.2.2/31 is directly connected, Ethernet6.2
 C        10.11.2.2/31 is directly connected, Ethernet5.2
 B E      10.128.0.0/16 [200/0] via 10.11.2.2, Ethernet5.2
 B E      10.129.0.1/32 [200/0] via VTEP 10.1.1.2 VNI 10002 router-mac 50:00:00:11:5d:47 local-interface Vxlan1
                                via VTEP 10.1.1.1 VNI 10002 router-mac 50:00:00:d0:a5:18 local-interface Vxlan1
 B E      10.129.0.2/32 [200/0] via VTEP 10.9.1.1 VNI 10002 router-mac 50:00:00:6b:0f:11 local-interface Vxlan1
                                via VTEP 10.9.1.2 VNI 10002 router-mac 50:00:00:5b:6f:f5 local-interface Vxlan1
 B E      10.129.0.0/16 [200/0] via 10.11.2.2, Ethernet5.2

```


