# Kroki do wykonania 10.2.4.4

## Konfiguracja

Swicze w pakiet tracerze nie maja wszystkich funkcji ipv6 wiec warto jest użyć 3650 Multilayer switch, aby sie odpalil musimy wmontowac mu zasilacz


### Konfiguracja R1

conf t -> hostname R1

```txt
ip domain name ccna-lab.com
ipv6 dhcp pool IPV6POOL-A
 dns-server 2001:DB8:ACAD:CAFE::A
 domain-name ccna-lab.com
interface g0/0
 no ip address
 shutdown
 duplex auto
 speed auto
interface g0/1
 no ip address
 duplex auto
 speed auto
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:A::11/64
 end
```

konfigurujemy stateless dhcp
show run

widzimy ze ipv6 dhcp pool jest poprawny

blad jest w interface GigabitEthernet0/1
w ipv6 address mamy ::11/64 zamiast ::1/64
conf t -> int g0/1 -> no ipv6 address 2001:db8:acad:a::11/64 -> ipv6 address 2001:db8:acad:a::1/64 -> no shut -> end

stateless dhcp musi zostac zkonfigurowane na interfejsie g0/1
conf t -> int g0/1 -> ipv6 dhcp server IPV6POOL-A -> ipv6 nd other-config-flag -> end

### Konfiguracja S1

conf t -> hostname S1

TEGO nie ladujemy nie chcemy wylaczonych interface plus podpinamy zupełnie inne 3650 ma tylko g a nie fa

```txt
interface range f0/1-24
 shutdown
interface range g0/1-2
 shutdown
interface Vlan1
 shutdown
end
```

tera robimy zeby vlan 1 uzywal tylko slaac
conf t -> int vlan 1 -> ipv6 address autoconfig -> no shut -> end

show ip inter brief
show ip inter vlan 1

tera widzimy ze pkt nie wspiera tej funkcjonalnosci

### Konfiguracja PC-A

w mojej wersji packet tracera to nie zadziala
