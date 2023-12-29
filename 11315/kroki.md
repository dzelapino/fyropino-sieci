# Kroki które należy wykonać w 11.3.1.5

Najpierw dodajemy hwici do routerow zeby podpiac seriala

## Konfiguracja urzadzen

### Konf PC-A

ipaddress 192.168.1.3
subnet 255.255.255.0
gateway 192.168.1.1

### Konf PC-B

ipaddress 192.168.1.4
subnet 255.255.255.0
gateway 192.168.1.1

### Pomijamy część step4 bo to to co zawsze

wbijamy do gateway

conf t -> hostname Gateway -> int g0/1 -> ip address 192.168.1.1 255.255.255.0 -> no shut -> int s0/0/1 -> ip address 209.165.200.225 255.255.255.252 -> no shut -> end

wbijamy do isp

conf t -> hostname ISP -> int s0/0/0 -> ip address 209.165.200.226 255.255.255.252 -> clock rate 128000 -> no shut -> int lo 0 -> ip address 198.133.219.1 255.255.255.255 -> end

### Teraz statyczny routing

wbijamy do ISP

conf t -> ip route 209.165.200.224 255.255.255.224 s0/0/0

nastepnie lecimy do Gateway

conf t -> ip route 0.0.0.0 0.0.0.0 s0/0/1

### Pora załadować konfiguracje do gateway

po conf t wklejamy

```txt
interface g0/1
 ip nat outside
 no shutdown
interface s0/0/0
 ip nat outside
interface s0/0/1
 no shutdown
ip nat inside source static 192.168.2.3 209.165.200.254
ip nat pool NAT_POOL 209.165.200.241 209.165.200.246 netmask 255.255.255.248
ip nat inside source list NAT_ACL pool NATPOOL
ip access-list standard NAT_ACL
 permit 192.168.10.0 0.0.0.255
banner motd $AUTHORIZED ACCESS ONLY$
end
```

## Troubleshooting statycznego NAT

w gateway z poziomu # wpisujemy: show ip nat translations potem show ip nat statistics
widzimy ze gitara w translations mamy 1 statica i w statystykach tez 1

w translations widzimy ze inside global jest poprawny natomiast inside local ma blad poniewaz powinien byc to adres PC-A czyli nie 192.168.2.3 a 192.168.1.3

wiec wyrzucamy linie z show running: ip nat inside source static 192.168.2.3 209.165.200.254

conf t -> no ip nat inside source static 192.168.2.3 209.165.200.254 -> ip nat inside source static 192.168.1.3 209.165.200.254 -> end

teraz patrzymy na interfejsy show running serial musi byc outside a gigabit musi byc inside
blad jest w gigabit poniewaz jest on outside zamiast inside
drugi blad jest w s0/0/0 poniewaz ma on ip nat outside
trzeci blad jest w serialu 0/0/1 brakuje mu ip nat outside

conf t -> int g0/1 -> no ip nat outside -> ip nat inside -> int s0/0/0 -> no ip nat outside -> int s0/0/1 -> ip nat outside -> end

teraz patrzymy akcje z PC-B
blad jest w ip access-list standard NAT_ACL
permitujemy 192.168.10.0 0.0.0.255 a powinnismy 192.168.1.0 0.0.0.255

conf t -> ip access-list standard NAT_ACL -> no 10 permit 192.168.10.0 0.0.0.255 ->
10 permit 192.168.1.0 0.0.0.255 -> end

tera patrzymy NAT_POOL prawdziwym ostatnim adresem bedzie 246 poniewaz 247 to broadcast
wiec pool jest git ale w nastepnej lini mamy blad poniewaz jest: ip nat inside source list NAT_ACL pool NATPOOL brakuje underscore

conf t -> no ip nat inside source list NAT_ACL pool NATPOOL -> ip nat inside source list NAT_ACL pool NAT_POOL -> end

## Testujemy

z PC-B pingujemy 198.133.219.1

z ISP pingujemy 209.165.200.254 gitara

w gateway show ip nat translations potem show ip nat statistics
widzimy ze 1 translacja jest statyczna a 9 dynamicznych

