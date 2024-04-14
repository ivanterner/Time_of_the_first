
Create GRE Tunnel

                                    ----------
                  10.10.10.0/24        ISP      20.20.20.0/24
                         .1     .10-----------.10     .100
                  --------.________________________ --------
172.16.0.0/16     L-WF     <---------------------->  R-WF     192.168.0.0/16 
                  --------.1   GRE 10.5.5.0/30     .2-------
                     

Создать файл для пооднятия тунеля на обоих серверах.
```console
/etc/gre.up
#L-WF
#!/bin/bash
ip tunnel add tun1 mode gre local 10.10.10.1 remote  20.20.20.100 ttl 255
ip addr add 10.5.5.1/30 dev tun1
ip link set tun1 up
#ip route add 192.168.0.0/16 via 10.5.5.2

/etc/gre.up
#R-WF
#!/bin/bash
ip tunnel add tun1 mode gre local 20.20.20.100 remote 10.10.10.1 ttl 255
ip addr add 10.5.5.2/30 dev tun1
ip link set tun1 up
#ip route add 172.16.0.0/16 via 10.5.5.1
```
Добавитьт в автозагрузку
```console
/etc/crontab
@reboot  root  /etc/gre.up
```
