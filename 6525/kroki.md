# Lab 6.5.2.5 po kolei

## W tabelce są errory

W HQ S0/0/1 ma ::1/30 a powinno być ::1/64

W HQ S0/0/0 ma ::20:1/64 powinno być :20::1/64

W ISP brakuje jedynki (jest ::/64) przed slaszem w S0/0/1

## Step 3 robi to co pozostałe laby więc pomijamy

## PCA

ipv4 192.168.0.3
subnet 255.255.255.0
default gateway 192.168.0.1
ipv6 2001:db8:acad::3 / 64
link local address FE80::2E0:8FFF:FE17:ED1B
ipv6 gateway fe80::1

## PCC

ipv4 192.168.1.3
subnet 255.255.255.0
default gateway 192.168.1.1
ipv6 2001:db8:acad:1::3 / 64
link local address tu wychodzi ze zawsze zostawiamy co jest
ipv6 gateway fe80::1

## WEB SERVER

ipv4 172.16.3.3
subnet 255.255.255.0
default gateway 172.16.3.1
ipv6 2001:db8:acad:30::3 / 64
link local address tu wychodzi ze zawsze zostawiamy co jest
ipv6 gateway fe80::1

## S1

## S3

## HQ

```txt
hostname HQ
ipv6 unicast-routing
interface GigabitEthernet0/1
 ipv6 address 2001:DB8:ACAD::1/64
 ip address 192.168.0.1 255.255.255.128
 ipv6 address FE80::1 link-local
interface Serial0/0/0
 ipv6 address 2001:DB8:ACAD:20::2/64
 ip address 10.1.1.2 255.255.255.252
 clock rate 800000
 no shutdown
interface Serial0/0/1
 ipv6 address 2001:DB8:ACAD:2::3/64
 ip address 192.168.0.253 255.255.255.252
 no shutdown
ip route 172.16.3.0 255.255.255.0 10.1.1.1
ip route 192.168.1.0 255.255.255.0 192.16.0.254
ipv6 route 2001:DB8:ACAD:1::/64 2001:DB8:ACAD:2::2
ipv6 route 2001:DB8:ACAD:30::/64 2001:DB8:ACAD::20:1
```

## BRANCH

```txt
hostname BRANCH
ipv6 unicast-routing
interface GigabitEthernet0/1
 ipv6 address 2001:DB8:ACAD:1::1/64
 ip address 192.168.1.1 255.255.255.0
 ipv6 address FE80::1 link-local
 no shutdown
 interface Serial0/0/0
 ipv6 address 2001:DB8:ACAD:2::2/64
 clock rate 128000
 ip address 192.168.0.249 255.255.255.252
 clock rate 128000
 no shutdown
ip route 0.0.0.0 0.0.0.0 10.1.1.2
ipv6 route ::/0 2001:DB8:ACAD::1
```

## ISP

```txt
hostname ISP
ipv6 unicast-routing
interface GigabitEthernet0/0
 ipv6 address 2001:DB8:ACAD:30::1/64
 ip address 172.16.3.11 255.255.255.0
 ipv6 address FE80::1 link-local
 no shutdown
interface Serial0/0/0
 ipv6 address 2001:DB8::ACAD:20:1/64
 ip address 10.1.1.1 255.255.255.252
 no shutdown
ip route 192.168.1.0 255.255.255.0 10.1.1.2
ipv6 route 2001:DB8:ACAD::/62 2001:DB8:ACAD:20::2
```

## Kolejne kroki

### Kroki HQ

widzimy ze gig0/1 w HQ jest disabled
podgladamy poprzez show running
trzeba go odpalic
conf t -> int g0/1 -> no shut -> end

nastepnie w show running widzimy ze Serial0/0/1 ma error niepoprawny adres ipv6 powinno byc 1/64 a jest 3/64
conf t -> int s0/0/1 -> no ipv6 address 2001:db8:acad:2::3/64 -> ipv6 address 2001:db8:acad:2::1/64 -> end

kolejny blad jest w ip route 192.168.1.0 255.255.255.0 192.16.0.254
sciezka do routera branch jest niepoprawna musimy zamienic 192.16.0.254 na 192.168.0.254
conf t -> no ip route 192.168.1.0 255.255.255.0 192.16.0.254 -> ip route 192.168.1.0 255.255.255.0 192.168.0.254 -> end

nastepny blad jest w ipv6 w sciezce do routera isp mamy niepoprawny ipv6 route 2001:DB8:ACAD:30::/64 2001:DB8:ACAD::20:1 <- zła kolejność dwukropków
conf t -> no ipv6 route 2001:db8:acad:30::/64 2001:db8:acad::20:1 -> ipv6 route 2001:db8:acad:30::/64 2001:db8:acad:20::1 -> end

### Kroki ISP

Pierwszy error widzimy w g0/0 zły ip address 172.16.3.11 powinno byc 172.16.3.1
conf t -> int g0/0 -> no ip address -> ip address 172.16.3.1 255.255.255.0 -> end

nastepny blad jest w serialu 0/0/0 niepoprawny ipv6 address 2001:DB8::ACAD:20:1/64 powinno być 2001:DB8:ACAD:20::1/64
conf t -> int s0/0/0 -> no ipv6 address 2001:db8::acad:20:1/64 -> ipv6 address 2001:db8:acad:20::1/64 -> end

nastepny error jest w ip route 192.168.1.0 255.255.255.0 10.1.1.2
powinno byc 192.168.0.0 255.255.254.0 10.1.1.2
conf t -> no ip route 192.168.1.0 255.255.255.0 10.1.1.2 -> ip route 192.168.0.0 255.255.254.0 10.1.1.2 -> end

### Kroki Branch

pierwszy error widzimy w Serialu 0/0/0 niepoprawny jest ip address 192.168.0.249 255.255.255.252
powinno byc 192.168.0.254 255.255.255.252
cont t -> int s0/0/0 -> no ip address -> ip address 192.168.0.254 255.255.255.252 -> end

nastepny blad jest w ip route 0.0.0.0 0.0.0.0 10.1.1.2 powinien tam być ip address next hopka czyli seriala 0/0/1 routera HQ
conf t -> no ip route 0.0.0.0 0.0.0.0 10.1.1.2 -> ip route 0.0.0.0 0.0.0.0 192.168.0.253 -> end

kolejny blad jest w ipv6 route ::/0 2001:DB8:ACAD::1
conf t -> no ipv6 route ::/0 2001:db8:acad::1 -> ipv6 route ::/0 2001:db8:acad:2::1 -> end

## Pingujemy z PCA

### Ping Web server

ping 172.16.3.3 -> ping 2001:db8:acad:30::3  

### Ping PCC

ping 192.168.1.3 -> ping 2001:db8:acad:1::3
