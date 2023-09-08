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



