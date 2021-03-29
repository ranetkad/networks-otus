<a id=top></a>

# DHCPv4

###  Топология:

![](scheme.PNG)

###  Таблица адресации:

| Device  |  Interface | IP Address  |   Subnet Mask| Gateway  |
| ------------ | ------------ | ------------ | ------------ | ------------ |
| R1  |  G0/0/0 | 10.0.0.1  | 255.255.255.252  | N/A  |
|   | GigabitEthernet0/0/1  | N/A  | N/A  |   |
|   |  GigabitEthernet0/0/1.100 | 192.168.1.1  | 255.255.255.192  |  N/A   |
|   | GigabitEthernet0/0/1.200  | 192.168.1.65  | 255.255.255.224  |   |
|   |  GigabitEthernet0/0/1.1000 | N/A  |  N/A  |   |
|  R2 |  G0/0/0| 10.0.0.2  | 255.255.255.252   | N/A  |
|   |  GigabitEthernet0/0/1 | 192.168.1.97  | 255.255.255.240  | N/A  |
|  S1 |  VLAN 200 | 192.168.1.66  | 255.255.255.224  | 192.168.1.65  |
|  S2 | VLAN 1  |  192.168.1.98 | 255.255.255.240  |   192.168.1.97 |
| PC-A  | NIC  | DHCP  | DHCP  | DHCP  |
|  PC-B |  NIC | DHCP  |  DHCP | DHCP  |


###  Таблица VLAN:

| VLAN  | Name  |  Interface Assigned |
| ------------ | ------------ | ------------ |
|  1 | N/A  |S2: F0/15 |
| 100  | Clients  |S1: F0/15   |
|  200 | Management  | S1: VLAN 200   |
| 999  | Parking_Lot  |  S1: F0/1-4, F0/7-24, G0/1-2 |
| 1000  | Native  | N/A  |

### Выдана подсеть 192.168.1.0/24.
       

            Подсеть А: Для клиентов R1, должна поддерживать 58 хостов. (192.168.1.1/26  с 1-63)

            Подсеть B: Для управления, должна поддерживать 28 хостоы. (192.168.1.65/27 с 66-95)

            Подсеть С:  Для клиентов R2, должна поддерживать 12 хостов. 192.168.1.97/28 c 97-111      


### Цели:

1. [Настройка основных параметров устройств](#1)
2. [Настроить два DHCPv4 сервера на R1 и проверить](#2)
3. [Настроить и проверить DHCP Relay  на  R2](#3)



###  Решение:
  1. Настройка основных параметров устройств:<a id=1></a>

      * Используем уже ранее готовый [файл](/lab/L02-DHCPv4/cfg/BasicDeviceSettings) с командами для  базовой настройки, меняя значения на нужные

      * Настройка Inter-VLAN Routing на R1 

            R1(config)# interface GigabitEthernet0/0/1.100
            R1(config-subif)# description clients
            R1(config-subif)# encapsulation dot1q 100
            R1(config-subif)# ip address 192.168.1.1 255.255.255.192
            R1(config-subif)# interface GigabitEthernet0/0/1.200
            R1(config-subif)# encapsulation dot1q 200
            R1(config-subif)# description mngmt
            R1(config-subif)# ip address 192.168.1.65 255.255.255.224
            R1(config-subif)# interface GigabitEthernet0/0/1.1000
            R1(config-subif)# encapsulation dot1q 1000 native
            R1(config-subif)# description native

            R1#sh ip interface  brief  | i 0/0/1
            GigabitEthernet0/0/1      unassigned      YES unset  up                    up 
            GigabitEthernet0/0/1.100  192.168.1.1     YES manual up                    up 
            GigabitEthernet0/0/1.200  192.168.1.65    YES manual up                    up 
            GigabitEthernet0/0/1.1000 unassigned      YES unset  up                    up 



      * Настройка GigabitEthernet0/0/1 на R2. Так же G0/0/0 и статические маршруты на обоих роутерах  

            R2(config)# interface g0/0/1
            R2(config-if)# ip address 192.168.1.97 255.255.255.240
            R2(config-if)# no shutdown          
            R2(config-if)# exit
            
            R1(config)# interface g0/0/0
            R1(config-if)# ip address 10.0.0.1 255.255.255.252
            R1(config-if)# no shutdown

            R2(config)# interface g0/0/0
            R2(config-if)# ip address 10.0.0.2 255.255.255.252
            R2(config-if)# no shutdown


            R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.2
            R2(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.1

            R1#ping 192.168.1.97
            Type escape sequence to abort.
            Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
            .!!!!
            Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/0 ms
      
      * Создание  VLAN на  S1 

            S1(config)# vlan 100
            S1(config-vlan)# name Clients
            S1(config-vlan)# vlan 200
            S1(config-vlan)# name mngmt
            S1(config-vlan)# vlan 999
            S1(config-vlan)# name Parking_Lot
            S1(config-vlan)# vlan 1000
            S1(config-vlan)# name native
            S1(config-vlan)# exit
      
      * Настройка  VSI на  S1 и S2

            S1(config)# interface vlan 200
            S1(config-if)# ip address 192.168.1.66 255.255.255.224
            S1(config-if)# no shutdown
            S1(config-if)# exit
            S1(config)# ip default-gateway 192.168.1.65

            S2(config)# interface vlan 1
            S2(config-if)# ip address 192.168.1.98 255.255.255.240
            S2(config-if)# no shutdown
            S2(config-if)# exit
            S2(config)# ip default-gateway 192.168.1.97

      * Назначения  портов  исходя из "Таблица VLAN"

            S1(config)# interface range f0/1–4, f0/7–24, g0/1–2
            S1(config-if-range)# switchport mode access
            S1(config-if-range)# switchport access vlan 999
            S1(config-if-range)# shutdown
            S1(config-if-range)# exit

            S2(config)# interface range f0/1–4,f0/6–17,f0/19–24,g0/1–2
            S2(config-if-range)# switchport mode access
            S2(config-if-range)# shutdown
            S2(config-if-range)# exit

            S1(config)# interface f0/15
            S1(config-if)# switchport mode access
            S1(config-if)# switchport access vlan 100

         
            S1(config-if)# switchport mode trunk
            S1(config-if)# switchport trunk native vlan 1000
            S1(config-if)# switchport trunk allowed vlan 100,200,1000

  2. Настройка  DCHPv4 серверов  на  R1 для Подсети А и В: <a id=2></a>

      * Исключить, первые пять адресов из каждого пула

            R1(config)#ip dhcp  excluded-address 192.168.1.1 192.168.1.5
      
      * Создать DHCP пул (Уникальное имя для каждого пула )

            R1(config)#ip dhcp pool R1_Client
      
      * Указать сеть, которая принадлежит DHCP 

            R1(dhcp-config)#network 192.168.1.0 255.255.255.192

      * Установить доменное имя ccna-lab.com

            R1(dhcp-config)#domain-name ccna-lab.com
      
      * Настройка шлюза DHCP

            R1(dhcp-config)#default-router 192.168.1.1
      
      * Настроить срок аренды 2 дня 12 часов и 30 минут 

            в PT  нельзя 
      
      * Так же  настроим  второй пул 

            R1(config)# ip dhcp excluded-address 192.168.1.97 192.168.1.101
            R1(config)# ip dhcp pool R2_Client_LAN
            R1(dhcp–config)# network 192.168.1.96 255.255.255.240
            R1(dhcp–config)# default-router 192.168.1.97
            R1(dhcp–config)# domain-name ccna-lab.com

      * Сохраним  конфигурацию 

            R1# copy running-config startup-config 


  3. Проверить конфигурацию DHCPv4:<a id=3></a>

        * R1#show ip dhcp pool

                  Pool R1_Client :
                  Utilization mark (high/low)    : 100 / 0
                  Subnet size (first/next)       : 0 / 0 
                  Total addresses                : 62
                  Leased addresses               : 0
                  Excluded addresses             : 5
                  Pending event                  : none
                  1 subnet is currently in the pool :
                  Current index        IP address range                    Leased/Excluded/Total
                  192.168.1.1          192.168.1.1      - 192.168.1.62      0     / 5     / 62  

                  Pool R2_Client_LAN :
                  Utilization mark (high/low)    : 100 / 0
                  Subnet size (first/next)       : 0 / 0 
                  Total addresses                : 14
                  Leased addresses               : 0
                  Excluded addresses             : 5
                  Pending event                  : none
                  1 subnet is currently in the pool :
                  Current index        IP address range                    Leased/Excluded/Total
                  192.168.1.97         192.168.1.97     - 192.168.1.110     0     / 5     / 14

        * show ip dhcp bindings 

                  R1#sh ip dhcp  binding 
                  Bindings from all pools not associated with VRF:
                  IP address      Client-ID/  Lease expiration  Type       State      Interface
                  Hardware address/
                  User name

        * show ip dhcp server statistics

                  R1#show ip dhcp server statistics
                  Memory usage         18951
                  Address pools        2
                  Database agents      0
                  Automatic bindings   0
                  Manual bindings      0
                  Expired bindings     0
                  Malformed messages   0
                  Secure arp entries   0
                  Renew messages       0
                  Workspace timeouts   0
                  Static routes        0
                  Relay bindings       0
                  Relay bindings active        0
                  Relay bindings terminated    0
                  Relay bindings selecting     0
                  Message              Received
                  BOOTREQUEST          0
                  DHCPDISCOVER         0
                  DHCPREQUEST          0
                  DHCPDECLINE          0
                  DHCPRELEASE          0
                  DHCPINFORM           0
                  DHCPVENDOR           0
                  BOOTREPLY            0
                  DHCPOFFER            0
                  DHCPACK              0
                  DHCPNAK              0
                  Message              Sent
                  BOOTREPLY            0
                  DHCPOFFER            0
                  DHCPACK              0
                  DHCPNAK              0
                  Message              Forwarded
                  BOOTREQUEST          0
                  DHCPDISCOVER         0
                  DHCPREQUEST          0
                  DHCPDECLINE          0
                  DHCPRELEASE          0
                  DHCPINFORM           0
                  DHCPVENDOR           0
                  BOOTREPLY            0
                  DHCPOFFER            0
                  DHCPACK              0
                  DHCPNAK              0
                  DHCP-DPM Statistics
                  Offer notifications sent        0
                  Offer callbacks received        0
                  Classname requests sent         0
                  Classname callbacks received    0

        * Попытка получить IP по DHCP на PC-A

                  C:\>ipconfig

                  FastEthernet0 Connection:(default port)

                  Connection-specific DNS Suffix..: ccna-lab.com
                  Link-local IPv6 Address.........: FE80::290:21FF:FE0A:61B5
                  IPv6 Address....................: ::
                  IPv4 Address....................: 192.168.1.6
                  Subnet Mask.....................: 255.255.255.192
                  Default Gateway.................: ::
                                                      192.168.1.1

        * Пинг до R1 G0/0/1

                  C:\>ping 192.168.1.1

                  Pinging 192.168.1.1 with 32 bytes of data:

                  Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
                  Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
                  Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
                  Reply from 192.168.1.1: bytes=32 time<1ms TTL=255

                  Ping statistics for 192.168.1.1:
                  Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
                  Approximate round trip times in milli-seconds:
                  Minimum = 0ms, Maximum = 0ms, Average = 0ms

  4. Настройка  и проверка  DHCP relay на  R2: <a id=4></a>

        * Настроить R2 в качестве DHCP-relay для G0/0/1    

                  R2(config)#interface g0/0/1
                  R2(config-if)#ip helper-address 10.0.0.1
        
        * Попытка получить IP по DHCP на PC-B

                  C:\>ipconfig

                  FastEthernet0 Connection:(default port)

                  Connection-specific DNS Suffix..: ccna-lab.com
                  Link-local IPv6 Address.........: FE80::2D0:BCFF:FE35:1873
                  IPv6 Address....................: ::
                  IPv4 Address....................: 192.168.1.102
                  Subnet Mask.....................: 255.255.255.240
                  Default Gateway.................: ::

        * Пинг до R1 G0/0/1

                  C:\>ping 192.168.1.1

                  Pinging 192.168.1.1 with 32 bytes of data:

                  Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
                  Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
                  Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
                  Reply from 192.168.1.1: bytes=32 time<1ms TTL=255

                  Ping statistics for 192.168.1.1:
                  Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
                  Approximate round trip times in milli-seconds:
                  Minimum = 0ms, Maximum = 0ms, Average = 0ms
        
      ------------

      [вверх ](#top)
      [конфиги ](/lab/L02-DHCPv4/cfg) 
      
            
                     
