# Kroki które należy wykonać

## PC-A

ip: 192.168.1.3
subnet: 255.255.255.0
gateway: 192.168.1.1

## PC-C

ip: 192.168.3.3
subnet: 255.255.255.0
gateway: 192.168.3.1

## Dodatki

Do routerow wkladamy hwic-2t

## S1

w cli wpisujemy
en -> conf t -> hostname S1 -> int vlan 1 -> ip address 192.168.1.11 255.255.255.0 -> no shut
-> exit -> ip default-gateway 192.168.1.1 -> end

## S3

w cli wpisujemy
en -> conf t -> hostname S3 -> int vlan 1 -> ip address 192.168.3.11 255.255.255.0 -> no shut
-> exit -> ip default-gateway 192.168.3.1 -> end

## Router ISP

w cli wpisujemy
en -> conf t -> hostname ISP

```txt
hostname ISP
interface GigabitEthernet0/1
 ip address 192.168.3.1 255.255.255.0
 no shutdown
interface Serial0/0/0
 ip address 10.1.1.1 255.255.255.252
 clock rate 128000
 no shutdown
router eigrp 1
 network 10.1.1.0 0.0.0.3
 network 192.168.3.0
 no auto-summary
end
```

## Router HQ

w cli wpisujemy
en -> conf t -> hostname HQ

### STEP 6

na forum piszą że w packet tracerze nie wszystko działa i ta rzecz to właśnie jedna z nich
dlatego hiszpan to zostawil

### wklejany config

```txt
hostname HQ
interface Loopback0
 ip address 192.168.4.1 255.255.255.0
interface GigabitEthernet0/1
 ip address 192.168.1.1 255.255.255.0
 ip access-group 101 out
 ip access-group 102 in
 no shutdown
interface Serial0/0/1
 ip address 10.1.1.2 255.255.255.252
 ip access-group 121 in
 no shutdown
router eigrp 1
 network 10.1.1.0 0.0.0.3
 network 192.168.1.0
 network 192.168.4.0
 no auto-summary
access-list 101 permit ip 192.168.11.0 0.0.0.255 any
access-list 101 deny ip any any
access-list 102 permit tcp any any established
access-list 102 permit icmp any any echo-reply
access-list 102 permit icmp any any unreachable
access-list 102 deny ip any any
access-list 121 permit tcp any host 192.168.4.1 eq 89
access-list 121 deny icmp any host 192.168.4.11
access-list 121 deny ip 192.168.1.0 0.0.0.255 any
access-list 121 deny ip 127.0.0.0 0.255.255.255 any
access-list 121 deny ip 224.0.0.0 31.255.255.255 any
access-list 121 permit ip any any
access-list 121 deny ip any any
end
```

## Part 2

### ACL 101

Wbijamy do hq
z poziomu # -> sh run -> widzimy ze access list 101 ma permit na 192.168.11.0 -> conf t -> no access-list 101 permit ip 192.168.11.0 0.0.0.255 any -> access-list 101 permit ip 192.168.1.0 0.0.0.255 any -> end

Potem musimy zrobić deny any to any
z poziomu # -> conf t -> access-list 101 deny ip any any -> end

Tera widzimy że kierunek acl 101 jest zły
z poziomu # -> conf t -> int g0/1 -> no ip access-group 101 out -> ip access-group 101 in -> end

### ACL 102

Znowu w hq
w running config patrzymy i w templatce
Extended ip access-lists 102
wszystko jest git

Teraz musimy położyć ACL 102 na interfejsie g0/1 żeby był aplikowany tylko do tego lan a nie wszystkich innych
w hq z # -> conf t -> int g0/1 -> ip access-group 102 out -> end

## Part 3

Wbijamy do hq
show running-config potem show access-list
musimy wywalić permit tcp any host 192.168.4.1 eq 89
lecimy z # -> conf t -> ip access-list extended 121 -> no 10 permit tcp any host 192.168.4.1 eq 89 -> end
tera podgladamy show access-lists i nie ma 10
nastepnie conf t -> ip access-list extended 121 -> 10 permit tcp any host 192.168.4.1 eq 80 -> end

kolejny blad jest w 20 mamy tam deny 192.168.4.11 a powinno byc 192.168.4.1

conf t -> ip access-list extended 121 -> no 20 deny icmp any host 192.168.4.11 -> 20 deny icmp any host 192.168.4.1 ->  end
podgladamy access-liste i wszystko jest cacy

teraz musimy postawić ACL 121 na serialu 0/0/1 tym z routera hq
lecimy show running-config
widzimy ze w Serial0/0/1 jest ip access-group 121 in
czyli wszystko jest bien

## Refleksja

### 1

Kolejnosc powinna być z specific do general ponieważ ACL jest wykonywany od góry do dołu w kolejności sekwencyjnej. Jeżeli warunek jest true to procesowanie listy jest zatrzymywane (nie patrzy dalej)

### 2

Zostanie zastosowany domyślny deny, powinniśmy uważać kiedy usuwamy ACL zaaplikowany do aktywnego interfejsu.
