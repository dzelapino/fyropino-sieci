# Kroki do konfiguracji topologi 10.1.4.4

## Konf S1

conf t -> hostname S1 -> int vlan 1 -> ip add 192.168.1.2 255.255.255.0 -> exit -> ip default-gateway 192.168.1.1

## Konf S2

conf t -> hostname S2 -> int vlan 1 -> ip add 192.168.0.2 255.255.255.128 -> exit -> ip default-gateway 192.168.0.1

## Konf R1

conf t -> hostname R1 -> int g0/0 -> ip add 192.168.0.1 255.255.255.128 -> no shut -> int g0/1 -> ip add 192.168.1.1 255.255.255.0 -> no shut -> int s0/0/0 -> ip add 192.168.0.253 255.255.255.252 -> clock rate 128000 -> no shut

## Konf R2

conf t -> hostname R2 -> int s0/0/0 -> ip add 192.168.0.254 255.255.255.252 -> no shut -> int s0/0/1 -> ip add 209.165.200.226 255.255.255.252 -> clock rate 128000 -> no shut

## Konf ISP

conf t -> hostname ISP -> int s0/0/1 -> ip add 209.165.200.225 255.255.255.252 -> no shutdown

## Konf EIGRP dla R1

z poziomu #
conf t -> router eigrp 1 -> network 192.168.0.0 0.0.0.127 -> network 192.168.0.252 0.0.0.3 -> network 192.168.1.0 0.0.0.255 -> no auto-summary

## Konf EIGRP dla R2

z poziomu #
conf t -> router eigrp 1 -> network 192.168.0.252 0.0.0.3 -> redistribute static -> exit -> ip route 0.0.0.0 0.0.0.0 209.165.200.225

## Konf summary static route dla ISP do sieci R1 i R2

conf t -> ip route 192.168.0.0 255.255.254.0 209.165.200.226

## Dodanie initial DHCP dla R1 i R2

### DHCP R1

conf t ->

```txt
interface GigabitEthernet0/1
 ip helper-address 192.168.0.253
```

### DHCP R2

conf t ->

```txt
ip dhcp excluded-address 192.168.11.1 192.168.11.9
ip dhcp excluded-address 192.168.0.1 192.168.0.9
ip dhcp pool R1G1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
ip dhcp pool R1G0
 network 192.168.0.0 255.255.255.128
 default-router 192.168.11.1
```

## Konfiguracja PC-A i PC-B

teraz PC-A i PC-B moga otrzymac swoje adresy przy pomocy dhcp
wbijamy desktop -> ip configuration i klikamy dhcp zamiast static
widzimy ze dhcp sie przewraca musimy znalezc blad

lecimy do R2 i sh run

default pool R1G1 jest git

default pool R1G0 ma niepoprawny default router 192.168.0.1 zamiast 192.168.11.1

conf t -> ip dhcp pool R1G0 -> default-router 192.168.0.1 -> end

nastepnie po sh run widzimy ze zle jeszcze jest ip dhcp excluded-address 192.168.11.1 192.168.11.9

powinna byc 1 zamiast 11

conf t -> no ip dhcp excluded-address 192.168.11.1 192.168.11.9 -> ip dhcp excluded-address 192.168.1.1 192.168.1.9 -> end

teraz lecimy do R1 -> sh run -> widzimy ze w interface GigabitEthernet0/1 ip helper-address ma zly adres powinno byc 192.168.0.254(ip interfejsu s0/0/0 z R2) zamiast 192.168.0.253
w GigabitEthernet0/0 brakuje nam ip helper-address

conf t -> int g0/0 -> ip helper-address 192.168.0.254 -> int g0/1 -> no ip helper-address 192.168.0.253 -> ip helper-address 192.168.0.254

teraz wszystko smiga
z PC-A i PC-B mozemy pingowac ISP -> ping 209.165.200.225
