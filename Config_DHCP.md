![Карта сети ](/dhcp.png)
```console
yum install dhcp-server.x86_64

cp /usr/share/doc/dhcp-server/dhcpd.conf.example /etc/dhcp/dhcpd.conf
cd /etc/dhcp/
vim dhcpd.conf

option domain-name "skill39.wsr";
option domain-name-servers 172.16.20.10;

default-lease-time 600;
max-lease-time 7200;

ddns-update-style interim;
ddns-updates on;

log-facility local7;

subnet 172.16.20.0 netmask 255.255.255.0 {
}
subnet 172.16.50.0 netmask 255.255.255.0 {
}
subnet 172.16.100.0 netmask 255.255.255.0 {
  range 172.16.100.65 172.16.100.75;
  option routers 172.16.100.1;
}

subnet 172.16.200.0 netmask 255.255.255.0 {
  range 172.16.200.65 172.16.200.75;
  option routers 172.16.200.1;
}

host l-cli-b {
  hardware ethernet 08:00:07:26:c0:a5;
  fixed-address 172.16.200.61;
}


vim /etc/default/isc-dhcp-server
INTERFACESv4="enp1s0 enp7s0"

systemctl start dhcpd.service
systemctl enable dhcpd.service 

```
