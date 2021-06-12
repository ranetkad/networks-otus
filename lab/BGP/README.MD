<a id=top></a>

#  eBGP

### Пролог:

  В данной домашней работе будет отображаться  настройка  только одного роутера  в  офисе  "Москва" где  будет описан каждый шаг. После чего условный  шаблон  можно использовать для настройки роутер в других офисах описанных ниже.  Вся  концепция сети будет находиться [тут](/lab/Net_IPv4_v6), что бы не нагружать лишний раз домашнюю работ

###  Задание Настроить  eBGP между офисами:

  1. Настроить  eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас;
  2. Настроить eBGP между провайдерами Киторн и Ламас;
  3. Настроить eBGP между Ламас и Триада;
  4. Настроить eBGP между офисом С.-Петербург и провайдером Триада;
  5. Проверить IP доступность между офисами Москва и С.-Петербург;


###  Решение:
  
  1. [Настроить  eBGP между офисами; Москва, Киторн, Ламас, Триада, С.-Петербург](#1)
  2. [Проверить IP доступность между офисами Москва и С.-Петербург](#5)


###  1. Настроить  eBGP между офисами; Москва, Киторн, Ламас, Триада, С.-Петербург  <a id=1></a>

  
  * Первым делом запустим  процесс bgp на R14 командой router bgp N, где N номер AS в офисе: 

        router bgp 1001 
  
  * Следующий наш шаг это указать наших соседей  R14 командой neighbor IPv4/6 remote-as N, где N номер AS соседа: 

        msk-R14(config-router)# neighbor 33.33.33.22 remote-as 101
        msk-R14(config-router)# neighbor 20FF:CCFF:200A:4::22 remote-as 101
         

    > Здесь  мы на R14 указываем  нашего соседа kit-R22 и номер его AS 101
    >> Для установления  соседства  по IPv6 требуется так же вести neighbor 20FF:CCFF:200A:4::22 activate

  * Теперь мы анонсируем сети который имеются у нас  в таблице  маршрутизации командой  network  IPv4/mask, IPv6/prefix(Предварительно  надо зайти в  address-family ipv6 unicast): 

        network 77.97.130.0 mask 255.255.255.0
        network 20FF:CCFF:2000:1::/64

  * Проверим  соседство; 

        msk-R14#sh ip bgp neighbors 33.33.33.22
        BGP neighbor is 33.33.33.22,  remote AS 101, external link
          BGP version 4, remote router ID 33.33.34.22
          BGP state = Established, up for 01:38:44
          Last read 00:00:27, last write 00:00:34, hold time is 180, keepalive interval is 60 seconds
          Neighbor sessions:
            1 active, is not multisession capable (disabled)

        msk-R14# sh ip bgp ipv6  unicast  neighbors 20FF:CCFF:200A:4::22
        BGP neighbor is 20FF:CCFF:200A:4::22,  remote AS 101, external link
          BGP version 4, remote router ID 33.33.34.22
          BGP state = Established, up for 01:39:47
          Last read 00:00:28, last write 00:00:28, hold time is 180, keepalive interval is 60 seconds
          Neighbor sessions:
            1 active, is not multisession capable (disabled)




###  2. Проверить IP доступность между офисами Москва и С.-Петербург  <a id=5></a>
  
  * Получается проверяем; 

          VPCS7> ip dhcp
          DDORA IP 192.168.1.12/24 GW 192.168.1.1
          VPCS7> ping 192.168.3.11
          84 bytes from 192.168.3.11 icmp_seq=1 ttl=58 time=3.207 ms
          84 bytes from 192.168.3.11 icmp_seq=2 ttl=58 time=2.050 ms
          84 bytes from 192.168.3.11 icmp_seq=3 ttl=58 time=2.137 ms
          84 bytes from 192.168.3.11 icmp_seq=4 ttl=58 time=2.039 ms
          84 bytes from 192.168.3.11 icmp_seq=5 ttl=58 time=2.102 ms
          VPCS7>

          VPCS8> ip dhcp
          DORA IP 192.168.3.11/24 GW 192.168.3.1
          VPCS8> ping 192.168.1.12
          84 bytes from 192.168.1.12 icmp_seq=1 ttl=57 time=1.960 ms
          84 bytes from 192.168.1.12 icmp_seq=2 ttl=57 time=1.941 ms
          84 bytes from 192.168.1.12 icmp_seq=3 ttl=57 time=2.190 ms
          84 bytes from 192.168.1.12 icmp_seq=4 ttl=57 time=2.186 ms
          84 bytes from 192.168.1.12 icmp_seq=5 ttl=57 time=2.138 ms
          VPCS8>





------------

[top](#top)
[конфиги ](/lab/BGP/cfg) 