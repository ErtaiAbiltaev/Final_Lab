# Final_Lab
1.Configure VLAN Create VLANs
D1-D2
vlan batch 50 111 112
A1-A2
vlan batch 43 50 111 112

2.Access and Trunk Port
D1 — uplink к C1 (g0/0/5), downlink к A1 (g0/0/1, g0/0/2)

int g0/0/5
port link-type trunk
port trunk allow-pass vlan 50 111 112
int g0/0/1
port link-type trunk
port trunk allow-pass vlan 50 111 112
int g0/0/2
port link-type trunk
port trunk allow-pass vlan 50 111 112

D2 — uplink к C1 (g0/0/5), downlink к A2 (g0/0/1, g0/0/2)

int g0/0/5
port link-type trunk
port trunk allow-pass vlan 50 111 112
int g0/0/1
port link-type trunk
port trunk allow-pass vlan 50 111 112
int g0/0/2
port link-type trunk
port trunk allow-pass vlan 50 111 112

A1 — uplink к D1/D2, PC1 (e0/0/1) vlan111, PC2 (e0/0/2) vlan112, AP1 (e0/0/22)

int g0/0/1
port link-type trunk
port trunk allow-pass vlan 50 111 112 43
int g0/0/2
port link-type trunk
port trunk allow-pass vlan 50 111 112 43
int e0/0/1
port link-type access
port default vlan 111
int e0/0/2
port link-type access
port default vlan 112
int e0/0/22
port link-type trunk
port trunk pvid vlan 43
port trunk allow-pass vlan 43 111 112

A2 — uplink к D1/D2, PC3 (e0/0/1) vlan111, PC4 (e0/0/2) vlan112, AP2 (e0/0/22)

int g0/0/1
port link-type trunk
port trunk allow-pass vlan 50 111 112 43
int g0/0/2
port link-type trunk
port trunk allow-pass vlan 50 111 112 43
int e0/0/1
port link-type access
port default vlan 111
int e0/0/2
port link-type access
port default vlan 112
int e0/0/22
port link-type trunk
port trunk pvid vlan 43
port trunk allow-pass vlan 43 111 112

3.Link Aggregation. Eth-Trunk
D1 — Eth-Trunk 1

int eth-trunk 1
mode lacp
port link-type trunk
port trunk allow-pass vlan 50 111 112
int g0/0/3
eth-trunk 1
int g0/0/4
eth-trunk 1

D2 — Eth-Trunk 1

int eth-trunk 1
mode lacp
port link-type trunk
port trunk allow-pass vlan 50 111 112
int g0/0/3
eth-trunk 1
int g0/0/4
eth-trunk 1

display eth-trunk 1
display eth-trunk 1 verbose

4.MSTP (Multiple Spanning Tree Protocol)
D1 — root для Instance 1 (VLAN 111)

stp mode mstp
stp region-configuration
region-name LAB
revision-level 1
instance 1 vlan 111
instance 2 vlan 112
active region-configuration
stp instance 1 priority 4096
stp instance 2 priority 8192

D2 — root для Instance 2 (VLAN 112)

stp mode mstp
stp region-configuration
region-name LAB
revision-level 1
instance 1 vlan 111
instance 2 vlan 112
active region-configuration
stp instance 1 priority 8192
stp instance 2 priority 4096

A1 и A2 — просто участники

stp mode mstp
stp region-configuration
region-name LAB
revision-level 1
instance 1 vlan 111
instance 2 vlan 112
active region-configuration

5.VRRP (Virtual Router Redundancy Protocol)
D1 — VLANIF + VRRP + Loopback

int loopback 0
ip add 50.3.3.3 255.255.255.255

int vlanif 50
ip add 10.1.50.101 255.255.255.0

int vlanif 111
ip add 172.16.111.1 255.255.255.0
vrrp vrid 1 virtual-ip 172.16.111.254
vrrp vrid 1 priority 120

int vlanif 112
ip add 172.16.112.1 255.255.255.0
vrrp vrid 2 virtual-ip 172.16.112.254
vrrp vrid 2 priority 100

D2 — VLANIF + VRRP + Loopback

int loopback 0
ip add 50.4.4.4 255.255.255.255

int vlanif 50
ip add 10.1.50.102 255.255.255.0

int vlanif 111
ip add 172.16.111.2 255.255.255.0
vrrp vrid 1 virtual-ip 172.16.111.254
vrrp vrid 1 priority 100

int vlanif 112
ip add 172.16.112.2 255.255.255.0
vrrp vrid 2 virtual-ip 172.16.112.254
vrrp vrid 2 priority 120

6.Single-Area OSPFv2
EdgeRT1

int loopback 0
ip add 50.1.1.1 255.255.255.255

int g0/0/0
ip add 10.1.1.101 255.255.255.252

int g0/0/1
ip add 192.168.137.254 255.255.255.0

int g0/0/2
ip add 172.16.128.1 255.255.255.0

ospf 1 router-id 50.1.1.1
area 0
network 10.1.1.100 0.0.0.3
network 172.16.128.0 0.0.0.255
network 50.1.1.1 0.0.0.0

ip route-static 0.0.0.0 0.0.0.0 192.168.137.1

C1

int loopback 0
ip add 50.2.2.2 255.255.255.255

int g0/0/0
ip add 10.1.1.102 255.255.255.252

int g0/0/1
ip add 10.1.1.106 255.255.255.252

int g0/0/2
ip add 10.1.1.110 255.255.255.252

int g0/0/3
ip add 10.10.10.1 255.255.255.0

ospf 1 router-id 50.2.2.2
area 0
network 10.1.1.100 0.0.0.3
network 10.1.1.104 0.0.0.3
network 10.1.1.108 0.0.0.3
network 10.10.10.0 0.0.0.255
network 50.2.2.2 0.0.0.0

D1 — uplink к C1 через vlanif50, OSPF

int vlanif 50
ip add 10.1.1.105 255.255.255.252

ospf 1 router-id 50.3.3.3
area 0
network 10.1.1.104 0.0.0.3
network 10.1.50.0 0.0.0.255
network 172.16.111.0 0.0.0.255
network 172.16.112.0 0.0.0.255
network 50.3.3.3 0.0.0.0

D2 — uplink к C1 через vlanif50, OSPF

int vlanif 50
ip add 10.1.1.109 255.255.255.252

ospf 1 router-id 50.4.4.4
area 0
network 10.1.1.108 0.0.0.3
network 10.1.50.0 0.0.0.255
network 172.16.111.0 0.0.0.255
network 172.16.112.0 0.0.0.255
network 50.4.4.4 0.0.0.0

DHCP Server (Router)

int loopback 0
ip add 50.5.5.5 255.255.255.255

int g0/0/0
ip add 10.10.10.67 255.255.255.0

ospf 1 router-id 50.5.5.5
area 0
network 10.10.10.0 0.0.0.255
network 50.5.5.5 0.0.0.0

7.DHCP
DHCP Server — пулыcopy

dhcp enable

ip pool vlan111
network 172.16.111.0 mask 255.255.255.0
gateway-list 172.16.111.254
dns-list 172.16.128.53
excluded-ip-address 172.16.111.1 172.16.111.10
lease day 1

ip pool vlan112
network 172.16.112.0 mask 255.255.255.0
gateway-list 172.16.112.254
dns-list 172.16.128.53
excluded-ip-address 172.16.112.1 172.16.112.10
lease day 1

ip pool vlan43
network 10.1.43.0 mask 255.255.255.0
gateway-list 10.1.43.1
dns-list 172.16.128.53
excluded-ip-address 10.1.43.1 10.1.43.100
lease day 1

int g0/0/0
dhcp select global

D1 — relay на DHCP Server

dhcp enable
int vlanif 111
dhcp select relay
dhcp relay server-ip 10.10.10.67

int vlanif 112
dhcp select relay
dhcp relay server-ip 10.10.10.67

D2 — relay на DHCP Server

dhcp enable
int vlanif 111
dhcp select relay
dhcp relay server-ip 10.10.10.67

int vlanif 112
dhcp select relay
dhcp relay server-ip 10.10.10.67

8.NAT (Easy IP)
EdgeRT1 — ACL + NAT outbound

acl 2000
rule 5 permit source 10.0.0.0 0.255.255.255
rule 10 permit source 172.16.0.0 0.0.255.255

int g0/0/1
nat outbound 2000

9.Remote Access (SSH, Telnet)
EdgeRT1 / C1 / D1 / D2 — один и тот же конфиг

aaa
local-user admin password cipher Admin@123
local-user admin privilege level 15
local-user admin service-type ssh telnet

stelnet server enable
ssh user admin authentication-type password
ssh user admin service-type stelnet
user-interface vty 0 4
authentication-mode aaa
protocol inbound all

10,DNS and HTTP
DNS Server — настройка в eNSP GUI

Двойной клик на устройстве "DNS Server"
Вкладка "Server" → выбрать "DNS Server"
Нажать "Start"
В поле "Domain Name" написать: www.lab.com
В поле "IP Address" написать: 172.16.128.80
Нажать "Add" → "Apply"
IP адрес DNS Server: 172.16.128.53/24
Gateway: 172.16.128.1

HTTP Server — настройка в eNSP GUI

Двойной клик на устройстве "HTTP Server"
Вкладка "Server" → выбрать "HTTP Server"
Нажать "Start"
Файл index.html уже есть по умолчанию
IP адрес HTTP Server: 172.16.128.80/24
Gateway: 172.16.128.1

Проверка с PC (HTTP Client) — в браузере eNSP

Двойной клик на PC1 (HTTP Client)
Вкладка "Browser"
В адресной строке написать: http://www.lab.com
или напрямую: http://172.16.128.80
Должна открыться страница
DNS relay на EdgeRT1 (опционально)copy

dns resolve
dns server 172.16.128.53

11.WLAN
AC1 — полный конфиг

int g0/0/0
ip add 10.10.40.24 255.255.255.0

dhcp enable
ip pool wlan_sta
network 10.1.43.0 mask 255.255.255.0
gateway-list 10.1.43.1
dns-list 172.16.128.53
excluded-ip-address 10.1.43.1 10.1.43.100
lease day 1

int g0/0/0
dhcp select global

wlan
ssid-profile name LAB_SSID
ssid LAB-WIFI

security-profile name LAB_SEC
security wpa2 psk pass-phrase Admin@123 aes

vap-profile name LAB_VAP
forward-mode tunnel
service-vlan vlan-id 43
ssid-profile LAB_SSID
security-profile LAB_SEC

ap-group name default
vap-profile LAB_VAP wlan 1 radio 0
vap-profile LAB_VAP wlan 1 radio 1

ospf 1 router-id 10.10.40.24
area 0
network 10.10.40.0 0.0.0.255

12.FTP
FTP Server — настройка в eNSP GUI

Двойной клик на устройстве "FTP Server"
Вкладка "Server" → выбрать "FTP Server"
Нажать "Start"
Запомни логин/пароль: admin / admin
(или посмотри в настройках сервера)
IP адрес FTP Server: 172.16.128.21/24
Gateway: 172.16.128.1
FTP с Huawei роутера (EdgeRT1 / C1)copy
ftp 172.16.128.21
(вводишь логин и пароль)
dir
get filename.txt
put vrpcfg.zip
bye
FTP с PC (в eNSP CLI на PC)copy
ftp 172.16.128.21
Username: admin
Password: admin
dir
get test.txt
bye
Huawei — FTP сервер на самом роутере (если нужно)copy

aaa
local-user ftpuser password cipher Ftp@123
local-user ftpuser privilege level 5
local-user ftpuser service-type ftp
local-user ftpuser ftp-directory flash:
ftp server enable

13.TFTP
Ubuntu — установка TFTP (bash)
sudo apt-get install -y tftpd-hpa

sudo nano /etc/default/tftpd-hpa
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/var/lib/tftpboot"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="--secure"

sudo mkdir -p /var/lib/tftpboot
sudo chmod 777 /var/lib/tftpboot
sudo systemctl restart tftpd-hpa
sudo systemctl enable tftpd-hpa
С Huawei роутера — отправить конфиг на TFTP
tftp 172.16.128.10 put vrpcfg.zip backup.zip
С Huawei роутера — скачать файл с TFTP
tftp 172.16.128.10 get backup.zip vrpcfg.zip

14.NTP

Ubuntu — установка NTP (bash)
sudo apt-get install -y ntp

sudo nano /etc/ntp.conf
server 127.127.1.0
fudge 127.127.1.0 stratum 2

sudo systemctl restart ntp
sudo systemctl enable ntp

Все Huawei устройства — NTP клиент (EdgeRT1, C1, D1, D2, A1, A2)

ntp-service unicast-server 172.16.128.10
ntp-service enable
