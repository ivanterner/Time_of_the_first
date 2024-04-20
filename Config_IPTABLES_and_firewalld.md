```console
#Настройка iptables
yum install iptables-services.noarch

iptables -t nat -A POSTROUTING -o enp1s0 -j MASQUERADE

iptables -t nat -A PREROUTING -p udp --dport 53 -i enp1s0 -j DNAT --to-destination 172.16.20.10

iptables-save >> /etc/sysconfig/iptables

systemctl enable iptables.service

systemctl start iptables.service

config
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [233:24784]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
COMMIT
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -o enp1s0 -j MASQUERADE
-A PREROUTING -i enp1s0 -p udp -m udp --dport 53 -j DNAT --to-destination 172.16.20.10
COMMIT


#Настройка firewalld
systemctl start firewalld.service 
systemctl enable firewalld.service 

firewall-cmd --permanent --zone=public --add-masquerade
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https
firewall-cmd --permanent --zone=public --add-service=ssh
firewall-cmd --permanent --zone=public --add-protocol=gre
firewall-cmd --permanent --zone=public --add-protocol=esp
firewall-cmd --permanent --zone=public --add-port=47/tcp

systemctl restart firewalld
firewall-cmd --zone=public --list-all

```
