# Kroki które należy wykonać

## Budujemy topologie

## Wrzucamy konfigi

### PCA

IP ADDRESS 192.168.10.3
SUBNET MASK 255.255.255.0
Default Gateway 192.168.10.1

### PCB

IP ADDRESS 192.168.20.3
SUBNET MASK 255.255.255.0
Default Gateway 192.168.20.1

### R1

-> Cli -> en -> configure terminal

```txt

hostname R1
enable secret class
no ip domain lookup
line con 0
 password cisco
 login
 logging synchronous
line vty 0 4
 password cisco
 login
interface loopback0
 ip address 209.165.200.225 255.255.255.224
interface gigabitEthernet0/1
 no ip address
interface gigabitEthernet0/1.1
 encapsulation dot1q 11
 ip address 192.168.1.1 255.255.255.0
interface gigabitEthernet0/1.10
 encapsulation dot1q 10
 ip address 192.168.11.1 255.255.255.0
interface gigabitEthernet0/1.20
 encapsulation dot1q 20
 ip address 192.168.20.1 255.255.255.0
end

```

### S1

-> Cli -> en -> conf t

```txt

hostname S1
enable secret class
no ip domain-lookup
line con 0
 password cisco
 login
 logging synchronous
line vty 0 15
 password cisco
 login
 vlan 10
 name R&D
 exit
interface fastethernet0/1
 switchport mode access
interface fastethernet0/5
 switchport mode trunk
interface vlan1
 ip address 192.168.1.11 255.255.255.0
ip default-gateway 192.168.1.1
end

```

potem conf t -> int g0/1 -> no shutdown

### S2

-> Cli -> en -> conf t

```txt

hostname S2
enable secret class
no ip domain-lookup
line con 0
 password cisco
 login
 logging synchronous
line vty 0 15
 password cisco
 login
vlan 20
 name Engineering
 exit
interface fastethernet0/1
 switchport mode trunk
interface fastethernet0/18
 switchport access vlan 10
 switchport mode access
interface vlan1
 ip address 192.168.1.12 255.255.255.0
ip default-gateway 192.168.1.1
end

```

## Po konfiguracji

Z Pca wbijamy do command prompt
ping 192.168.20.3
Nie przechodzi
tracert 192.168.20.3
ping 192.169.10.1
Też nie działa
Czyli Switch 1 jest źle podpięty
Wbijamy do S1

show run

Swicz uno powinien mieć mode Trunk na 0/1 a ma mode access

Wklepujemy

conf t -> int fa0/1 -> switchport mode trunk -> end

ogolnie f0/6 ma trunka więc to tez trzeba zmienic

conf t -> int fa0/6 -> no switchport mode trunk -> switchport mode access  -> switchport access vlan 10 -> end

W S1 mozna wpisac show vlan brief, potem show interface trunk

wpisujemy conf -> int vlan 1 -> no shutdown -> end

Tera z PCA pingujemy 192.168.10.1

Nie smiga wiec problem jest w R1

Wbijamy do R1

Hasło -> en -> Hasło -> show running config -> conf t -> int g0/1.1 -> encapsulation dot1Q 1 -> int g0/1.10 -> ip add 192.168.10.1 255.255.255.0 -> end

Tera do PCa i pingujemy 192.168.10.1
Gitara

Tera z PCa pingujemy 192.168.1.11
Gitara

Ping 192.168.1.12
Nie dziala

Wbijamy do S2
Hasło -> en -> Hasło -> show running config ->  end -> configure terminal -> int fa0/18 -> switchport access vlan 20 -> exit -> int vlan 1 -> no shutdown -> end -> show vlan brief -> conf t -> no vlan 10 -> exit

tera z PCA ping 192.168.10.1 potem ping 192.168.1.11
nastepnie pingujemy 192.168.1.12 ostatecznie ping 192.168.20.3
