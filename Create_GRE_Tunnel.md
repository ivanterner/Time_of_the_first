![Карта сети ](/img/gre.png)             


Содать файлы на серверах /etc/gre.up

L-WF
```yaml
#!/bin/bash
ip tunnel add tun1 mode gre local 10.10.10.1 remote  20.20.20.100 ttl 255
ip addr add 10.5.5.1/30 dev tun1
ip link set tun1 up
ip route add 192.168.0.0/16 via 10.5.5.2
```
R-WF
```yaml
#!/bin/bash
ip tunnel add tun1 mode gre local 20.20.20.100 remote 10.10.10.1 ttl 255
ip addr add 10.5.5.2/30 dev tun1
ip link set tun1 up
ip route add 172.16.0.0/16 via 10.5.5.1
```

Добавить в автозагрузку /etc/crontab
```yaml
@reboot  root  /etc/gre.up
```
