acl 2000
 rule 5 permit source 172.16.111.0 0.0.0.255
 rule 10 permit source 172.16.112.0 0.0.0.255
 rule 15 permit source 172.16.128.0 0.0.0.255
 rule 20 permit source 10.1.50.0 0.0.0.255
 rule 25 permit source 10.1.1.0 0.0.0.255
 rule 30 permit source 192.168.100.0 0.0.0.255
 rule 35 permit source 192.168.200.0 0.0.0.255
 
quit

interface G0/0/1
 nat outbound 2000
quit

ip route-static 0.0.0.0 0.0.0.0 192.168.137.1

ospf 1
 area 0
  network 192.168.137.0 0.0.0.255
 default-route-advertise always
quit
