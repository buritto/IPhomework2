# Карамов Артур КН-202 n=12

## Шаг 0
В сети настроим маршрутизаторы. Команды будут в целом похожи, поэтому приведём пример для R2AR0:
``` R2AR0>enable 
R2AR0#conf t
R2AR0(config)#int fa0/0
R2AR0(config-if)#ip address 10.2.2.2 255.255.255.0
R2AR0(config-if)#no shutdown 
R2AR0(config-if)#exit
R2AR0(config)#int Serial 2/0
R2AR0(config-if)#ip address 10.3.3.1 255.255.255.0
R2AR0(config-if)#no shutdown 
R2AR0(config-if)#exit
R2AR0(config)#int loopback 0
R2AR0(config-if)#ip address 10.22.0.1 255.255.255.255
R2AR0(config-if)#no shutdown 
R2AR0(config-if)#exit
R2AR0(config)#int loopback 1
R2AR0(config-if)#ip address 10.22.1.1 255.255.255.255
R2AR0(config-if)#no shutdown 
R2AR0(config-if)#exit
R2AR0(config)#int loopback 2
R2AR0(config-if)#ip address 10.22.2.1 255.255.255.255
R2AR0(config-if)#no shutdown 
R2AR0(config-if)#exit
R2AR0(config)#int loopback 3
R2AR0(config-if)#ip address 10.22.3.1 255.255.255.255
R2AR0(config-if)#no shutdown 
R2AR0(config-if)#exit
Router(config)#router ospf 1
Router(config-router)#router-id 2.2.2.2
Router(config-router)#exit
Router(config)#exit
Router#clear ip ospf process
Router#wr mem
```
## Шаг1
Запустим ospf на маршрутизаторах 

"Почему для R2AR0 сеть для loopback интерфейсов задана в таком виде?"
__Ответ:__ такая маска позволит захватить все сети, которые эмулируют loopback'и
```
R1AR0(config)#router ospf 1
R1AR0(config-router)#network 10.1.1.0 0.0.0.255 area 0
R1AR0(config-router)#network 10.2.2.0 0.0.0.255 area 0

R2AR0(config)#router ospf 1
R2AR0(config-router)#network 10.2.2.0 0.0.0.255 area 0
R2AR0(config-router)#network 10.22.0.0 0.0.255.255 area 0

ABR(config)#router ospf 1
ABR(config-router)#network 10.0.0.0 0.255.255.255 area 0
```
## Шаг2
```
R1AR0#show ip route

    10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
C       10.1.1.0/24 is directly connected, Serial2/0
C       10.2.2.0/24 is directly connected, FastEthernet0/0
O       10.22.0.1/32 [110/2] via 10.2.2.2, 00:02:02, FastEthernet0/0
O       10.22.1.1/32 [110/2] via 10.2.2.2, 00:02:02, FastEthernet0/0
O       10.22.2.1/32 [110/2] via 10.2.2.2, 00:02:02, FastEthernet0/0
O       10.22.3.1/32 [110/2] via 10.2.2.2, 00:02:02, FastEthernet0/0
O IA 192.168.1.0/24 [110/128] via 10.1.1.2, 00:08:18, Serial2/0
O IA 192.168.2.0/24 [110/192] via 10.1.1.2, 00:08:18, Serial2/0 
```
Как интерпретировать «10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks»?
__Ответ:__ Маршрутизатор знает о 6 подсетях с таким префиксом, при этом под такой префикс подходят маски 2 сетей

## Шаг3
```
R1AR0#show ip ospf database router

            OSPF Router with ID (1.1.1.1) (Process ID 1)

                Router Link States (Area 0)

  LS age: 872
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 1.1.2.2
  Advertising Router: 1.1.2.2
  LS Seq Number: 80000003
  Checksum: 0x8415
  Length: 48
  Area Border Router
  Number of Links: 2

    Link connected to: another Router (point-to-point)
     (Link ID) Neighboring Router ID: 1.1.1.1
     (Link Data) Router Interface address: 10.1.1.2
      Number of TOS metrics: 0
       TOS 0 Metrics: 64

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.1.1.0
     (Link Data) Network Mask: 255.255.255.0
      Number of TOS metrics: 0
       TOS 0 Metrics: 64

  LS age: 838
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 1.1.1.1
  Advertising Router: 1.1.1.1
  LS Seq Number: 80000005
  Checksum: 0xc9a2
  Length: 60
  Number of Links: 3

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 10.2.2.2
     (Link Data) Router Interface address: 10.2.2.1
      Number of TOS metrics: 0
       TOS 0 Metrics: 1

    Link connected to: another Router (point-to-point)
     (Link ID) Neighboring Router ID: 1.1.2.2
     (Link Data) Router Interface address: 10.1.1.1
      Number of TOS metrics: 0
       TOS 0 Metrics: 64

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.1.1.0
     (Link Data) Network Mask: 255.255.255.0
      Number of TOS metrics: 0
       TOS 0 Metrics: 64

  LS age: 492
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 2.2.2.2
  Advertising Router: 2.2.2.2
  LS Seq Number: 8000000e
  Checksum: 0x7ccc
  Length: 84
  Number of Links: 5

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 10.2.2.2
     (Link Data) Router Interface address: 10.2.2.2
      Number of TOS metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.22.0.1
     (Link Data) Network Mask: 255.255.255.255
      Number of TOS metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.22.1.1
     (Link Data) Network Mask: 255.255.255.255
      Number of TOS metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.22.2.1
     (Link Data) Network Mask: 255.255.255.255
      Number of TOS metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.22.3.1
     (Link Data) Network Mask: 255.255.255.255
      Number of TOS metrics: 0
       TOS 0 Metrics: 1
```
1. Какой тип этого LSA?

2. Сколько Router LSA хранится на хранится на R1AR0? Почему именно столько? А сколько
Router LSA хранится на других маршрутизаторах этой области?

3. Кто генерирует Router LSA? Какую информацию содержат Router LSA? В каких пределах
распространяются маршрутизаторами Router LSA (в пределах сети, области, автономной
системы и т. п.)?

__Ответ:__
1. Router LSA
2. 3, потому что он имеет 2 соседей + информация о себе, на оставшихся по 2
3. Сам маршрутизатор. Информацию о соседях внутри области. В переделах области

## Шаг4
Используя команду show ip ospf database network выведем информацию о Network
LSA, хранящихся на R1AR0.
```
R2AR0#show ip ospf database network

            OSPF Router with ID (2.2.2.2) (Process ID 1)

                Net Link States (Area 0)

  Routing Bit Set on this LSA
  LS age: 230
  Options: (No TOS-capability, DC)
  LS Type: Network Links
  Link State ID: 10.2.2.2  (address of Designated Router)
  Advertising Router: 2.2.2.2
  LS Seq Number: 80000002
  Checksum: 0x3167
  Length: 32
  Network Mask: /24
        Attached Router: 1.1.1.1
        Attached Router: 2.2.2.2
```	
1. Какой тип этого LSA?
2. Сколько Network LSA хранится на хранится на R1AR0? Почему именно столько? А
сколько Network LSA хранится на других маршрутизаторах этой области?
3. Что такое Designated Router? Зачем он нужен?
4. Кто генерирует Network LSA? Какую информацию содержат Network LSA? В каких пре-
делах распространяются маршрутизаторами Network LSA?

__Ответ:__
1.Network LSA
2.1, потому что выделен один Designated Router с адресом 10.2.2.2 , на других также по 1
3.Выделенный роутер, который занимается рассылкой LSA по сети.
4.Designated Router, информацию о местонахождении Designated, внутри области.

## Шаг 5
В зоне 1 проделываем схожую манипуляции из Шаг1 и Шаг2, основываясь на информацие из табличек.

## Шаг6
```
R1AR0#show ip route

     10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
C       10.1.1.0/24 is directly connected, Serial2/0
C       10.2.2.0/24 is directly connected, FastEthernet0/0
O       10.22.0.1/32 [110/2] via 10.2.2.2, 00:42:31, FastEthernet0/0
O       10.22.1.1/32 [110/2] via 10.2.2.2, 00:42:31, FastEthernet0/0
O       10.22.2.1/32 [110/2] via 10.2.2.2, 00:42:31, FastEthernet0/0
O       10.22.3.1/32 [110/2] via 10.2.2.2, 00:42:31, FastEthernet0/0
O IA 192.168.1.0/24 [110/128] via 10.1.1.2, 00:48:47, Serial2/0
O IA 192.168.2.0/24 [110/192] via 10.1.1.2, 00:48:47, Serial2/0


R1AR0#show ip ospf data
            OSPF Router with ID (1.1.1.1) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
1.1.1.1         1.1.1.1         1329        0x80000006 0x00c7a3 3
1.1.2.2         1.1.2.2         1363        0x80000004 0x008216 2
2.2.2.2         2.2.2.2         984         0x8000000f 0x007acd 5

                Net Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum
10.2.2.2        2.2.2.2         1331        0x80000002 0x003167

                Summary Net Link States (Area 0)
Link ID         ADV Router      Age         Seq#       Checksum
192.168.1.0     1.1.2.2         1358        0x80000003 0x00208c
192.168.2.0     1.1.2.2         1355        0x80000004 0x0095d4
```
1.В таблице маршрутизации появились маршруты помеченные кодом «O IA». Что означает
этот код?
2. Сколько сетей передается из Area 1 в Area 0? Какие? Сколько Summary LSA хранится на
R1AR0?
3. Кто и каким образом генерирует Summary LSA? Какую информацию содержат Summary
LSA?

__Ответ:__
1. Маршрутизаторы из другой области.
2.2:192.168.1.0 
    192.168.2.0, также 2 для каждого маршрутизатора
3. Генерирует ABR. Описывает сети других областей.


## Шаг7

Подключим к Area1 маршрутизатор Router-PT RIP. На нём зададим ip адресса на loopback интерфейсах.
Настроим протокол RIP на этом маршрутизаторе:
```
RIP(config)#router rip
RIP(config-router)#version 2
RIP(config-router)#no auto-summary 
RIP(config-router)#network 10.0.0.0
RIP(config-router)#network 172.20.0.0
RIP(config-router)#exit

На R2AR0 также запустим протокол RIP и синхронизируем его с ospf 

R2AR0(config)#router rip
R2AR0(config-router)#version 2
R2AR0(config-router)#network 10.0.0.0
R2AR0(config-router)#ext
R2AR0(config-router)#exit
R2AR0(config)#router ospf 1
R2AR0(config-router)#redistribute rip subnets 


R1AR0#sh ip ro

     10.0.0.0/8 is variably subnetted, 7 subnets, 2 masks
C       10.1.1.0/24 is directly connected, Serial2/0
C       10.2.2.0/24 is directly connected, FastEthernet0/0
O E2    10.3.3.0/24 [110/20] via 10.2.2.2, 00:08:44, FastEthernet0/0
O       10.22.0.1/32 [110/2] via 10.2.2.2, 00:11:23, FastEthernet0/0
O       10.22.1.1/32 [110/2] via 10.2.2.2, 00:11:23, FastEthernet0/0
O       10.22.2.1/32 [110/2] via 10.2.2.2, 00:11:23, FastEthernet0/0
O       10.22.3.1/32 [110/2] via 10.2.2.2, 00:11:23, FastEthernet0/0
     172.20.0.0/24 is subnetted, 3 subnets
O E2    172.20.0.0 [110/20] via 10.2.2.2, 00:08:02, FastEthernet0/0
O E2    172.20.1.0 [110/20] via 10.2.2.2, 00:08:02, FastEthernet0/0
O E2    172.20.2.0 [110/20] via 10.2.2.2, 00:08:02, FastEthernet0/0
```
1.В таблице маршрутизации появились записи с кодом «O E2».Что они означают?
2. Объяснить смысл данных выводимых командой show ip ospf database
external.
3. Кто и каким образом генерирует External LSA? Какую информацию содержат External
LSA?

__Ответ:__
1.Сети вне нашей области, используется стоимость внешнего маршрута.
2.Перед нами таблица в которой можно проследить к какой зоне пренадлежит тот или иной маршрутизатор
3.Генерирует R2AR0 так как подключён в внешнему домену маршрутизации и вынужден передавать чужие маршруты в ospf.

## Шаг8
На всех маршрутизаторах area1 выполняем:
```
router ospf 1
area 1 stub
```
```
R1AR1#show ip route 

     10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O IA    10.1.1.0/24 [110/128] via 192.168.1.1, 00:27:25, Serial2/0
O IA    10.2.2.0/24 [110/129] via 192.168.1.1, 00:27:25, Serial2/0
O IA    10.22.0.1/32 [110/130] via 192.168.1.1, 00:26:45, Serial2/0
O IA    10.22.1.1/32 [110/130] via 192.168.1.1, 00:26:45, Serial2/0
O IA    10.22.2.1/32 [110/130] via 192.168.1.1, 00:26:45, Serial2/0
O IA    10.22.3.1/32 [110/130] via 192.168.1.1, 00:26:45, Serial2/0
C    192.168.1.0/24 is directly connected, Serial2/0
C    192.168.2.0/24 is directly connected, Serial3/0
O*IA 0.0.0.0/0 [110/65] via 192.168.1.1, 00:27:25, Serial2/0
```
1.Что такое тупиковая область? Для чего нужна?
2. Как отразилось на таблице маршрутизации и содержимом топологической базы маршру-
тизаторов ABR и R1AR1 то, что область стала тупиковой?
3. Предположим, что из области Area 1 понадобилось передать в OSPF какой-то внешний
маршрут, например, пробросить статический маршрут до провайдера и т.д. В топологии в
Area 1 есть роутер ASBR, на котором настроено несколько статических маршрутов. Уда-
ется ли передать эти маршруты в OSPF? Почему?

__Ответ:__
1. Тупиковая область - это такая область через каторую ospf не будет передавать External-LSA.
2.Для AS-external маршрутов появился 1 маршрут по умолчанию
3. Тупиковая область не разрешит такой маршрут, так как он является External LSA
