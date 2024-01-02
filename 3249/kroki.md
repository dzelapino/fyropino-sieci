# Kroki do wykonania 3.2.4.9

## Konfiguracja

### Konf PC-a

ip 192.168.10.2
sub 255.255.255.0
gateway 192.168.10.1

### konf PC-B

ip 192.168.10.3
sub 255.255.255.0
gateway 192.168.10.1

### Konf PC-C

ip 192.168.20.3
sub 255.255.255.0
gateway 192.168.20.1

### Konf S1

```txt
hostname S1
vlan 10
name Students
vlan 2
!vlan 20
name Faculty
vlan 30
name Guest
interface range f0/1-24
switchport mode access
shutdown
interface range f0/7-12
switchport access vlan 10
interface range f0/13-18
switchport access vlan 2
interface range f0/19-24
switchport access vlan 30
end
```

### Konf S2

```txt
hostname S2
vlan 10
Name Students
vlan 20
Name Faculty
vlan 30
Name Guest
interface f0/1
switchport mode trunk
switchport trunk allowed vlan 1,10,2,30
interface range f0/2-24
switchport mode access
shutdown
interface range f0/13-18
switchport access vlan 20
interface range f0/19-24
switchport access vlan 30
shutdown
end
```

## Po wklepaniu danych

widzimy ze Fa01 musi byc trunk
w s1 wpisujemy show interfaces trunk i widzimy ze jest pusto

conf t -> int fa0/1 -> switchport mode trunk -> no shutdown ->  end

nastepnie widzimy ze interfejsy 0/6-12 musz byc w vlan 10, brakuje interfejsu 6

conf t -> int f0/6 -> switchport mode access -> switchport access vlan 10 -> no shutdown -> end

lecimy show vlan brief
nie ma vlan 20 (jest 2 zamiast 20)

conf t -> int range fa0/13-18 -> switchport mode access -> exit -> vlan 20 -> exit -> no vlan 2 -> vlan 20 -> name Faculty -> exit -> int range f0/13-18 -> switchport mode access -> switchport access vlan 20 -> end

teraz patrzymy show running czy wszystko jest gitara
w vlan1 brakuje ip address

conf t -> int vlan 1 -> ip address 192.168.1.2 255.255.255.0 -> no shutdown -> end

bajera

## teraz pora na S2

show vlan brief

vlan 10 jest popsuty, nie ma portow

conf t -> int range fa0/6-12 -> switchport mode access -> switchport access vlan 10 -> end -> conf t -> int fa0/11 -> no shutdown -> end

po wpisaniu show vlan brief widzimy ze jest ok

tera show interfaces trunk
vlan 1 jest git
musimy odpalic fa0/18

conf t -> int fa0/18 -> no shutdown -> end

show running i widzimy ze vlan 1 nie ma adresu ip

conf t -> int vlan 1 -> ip address 192.168.1.3 255.255.255.0 -> no shut -> end

sh run i widzimy ze interface FastEthernet pozwala na vlan 1-2,10,30 brakuje 20 i jest 2

conf t -> int fa0/1 -> no switchport trunk allowed vlan 1-2,10,30 -> switchport trunk allowed vlan 1,10,20,30 -> end

sh run, show interfaces trunk

## pingowanie

z pc-a pingujemy 192.168.10.3
pingi z pc-a i pc-b do pc-c nie przejda poniewaz pc-c jest w innej sieci
