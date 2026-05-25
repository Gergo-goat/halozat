VLAN beállítás:
enable
configure terminal

vlan 103
 name iroda103
exit

interface FastEthernet0/1
 switchport mode access
 switchport access vlan 103
 exit

interface FastEthernet0/24
 switchport mode trunk
 exit

Al-interfész
enable
configure terminal

interface GigabitEthernet0/0
 no shutdown
 exit

interface GigabitEthernet0/0.103
 encapsulation dot1Q 103
 ip address 10.0.103.1 255.255.255.0
 no shutdown
 exit

DHCP:
enable
configure terminal

ip dhcp excluded-address 10.0.103.1 10.0.103.10

ip dhcp pool iroda103
 network 10.0.103.0 255.255.255.0
 default-router 10.0.103.1
 dns-server 10.0.254.254
 exit

OSPF:
router ospf 10
 network 10.0.103.0 0.0.0.255 area 0
 passive-interface GigabitEthernet0/0.103
 exit


PPP + CHAP:
username KOZPONT-RTR password vizsga123

interface Serial0/0/0
 encapsulation ppp
 ppp authentication chap
 no shutdown
 exit

Wireless:
ssid ----- pl: iroda_vendeg
authentication ------ WPA2-PSK
PSK pass phrase ---- pl:vendeg99
--kliens csatlakoztatás--
pc- desktop- pc wireless

NAT + PAT:
! NAT PAT beállítása a külső interfészre
ip nat inside source list 1 interface Serial0/0/1 overload

! Interfészek megjelölése (ha még nincs beállítva)
interface Serial0/0/1
 ip nat outside
 exit

interface Serial0/0/0
 ip nat inside
 exit

ACL:
ACL a belső hálózathoz (10.0.0.0/8 teljes tartomány)
access-list 1 permit 10.0.0.0 0.255.255.255

A router egy új hálózattal fog bővülni, amely a loopback0 interfészen keresztül lesz elérhető.
Az új alhálózat a 192.168.50.0/24 hálózat két egyenlő részre bontásából létrejövő 2. alhálózat
lesz. Állítsa be a loopback0 interfészen az alhálózat utolsó még használható IP-címét!:
Számítás:
192.168.50.0/24  →  /25-re osztva:
  1. alhálózat:  192.168.50.0/25   (cím: .0, broadcast: .127)
  2. alhálózat:  192.168.50.128/25 (cím: .128, broadcast: .255)

Utolsó használható IP: 192.168.50.254
Alhálózati maszk /25: 255.255.255.128

enable
configure terminal

interface Loopback0
 ip address 192.168.50.254 255.255.255.128
 no shutdown
 exit

Konfiguráljon a switch routerhez csatlakozó portján portbiztonságot. Legfeljebb 2 MAC-cím
megtanulását engedélyezze a porton. Állítsa be azt a funkciót, hogy a megtanult MAC-címek
ne csak a címtáblába, de a konfigurációba is kerüljenek bele automatikusan. Válasszon egy
olyan működési módot, amely azt eredményezi, hogy a portbiztonság megsértésekor nem
kapcsol le a port.:
telnet 192.168.0.2

enable
configure terminal

interface FastEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 exit

 EtherChannel / Port-aggregáció (PAgP):
 interface range FastEthernet0/1 - 2
 channel-group 1 mode desirable

 ! A portcsatorna trunk módra állítása
interface Port-channel1
 switchport mode trunk
 exit

 Alapértelmezett útvonal + OSPF:
 ip route 0.0.0.0 0.0.0.0 Serial0/0/1
 router ospf 10
  default-information originate
  exit

SSH konfiguráció
router# configure terminal

! Tartománynév beállítása (RSA kulcshoz szükséges)
ip domain-name jedlik.eu

! Felhasználó létrehozása
username rtradmin privilege 15 secret adminpswd

! RSA kulcs generálása (1024 bit)
crypto key generate rsa
 ! Kérdésre: 1024

! SSH v2 engedélyezése
ip ssh version 2

! VTY vonal beállítása SSH-ra
line vty 0 4
 login local
 transport input ssh
 exit


IPv6 konfiguráció – fizikai switch:
switch# configure terminal

! IPv6 unicast routing engedélyezése (ha szükséges)
ipv6 unicast-routing

! VLAN1 interfészen IPv6 cím beállítása
interface Vlan1
 ipv6 address 2001:DB8:13C::1/64
 ipv6 address FE80::1 link-local
 no shutdown
 exit

 Amennyiben az Ön által használt eszköz nem támogatja az IPv6 protokoll használatát, akkor
határozza meg a 192.168.99.0/24 hálózat második 32 méretű hálózatának utolsó
használható állomás IP-címét és állítsa be azt a 99-es VLAN interfész címének (a VLAN-t nem
szükséges létrehoznia). Állítsa be a hálózat első IP-címét alapértelmezett átjárónak.:

switch(config)# interface Vlan99
switch(config-if)# ip address 192.168.99.1 255.255.255.255
switch(config-if)# no shutdown
switch(config-if)# exit

switch(config)# ip default-gateway 192.168.99.0


