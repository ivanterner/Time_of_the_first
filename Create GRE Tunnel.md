
Create GRE Tunnel
      
                  --------.1    GRE      .2--------
       20.20.20.100 L-WF   <------------>   R-WF 10.10.10.1
                  --------   10.5.5.1/30   --------
                     



/etc/gre.up

#L-WF

#!/bin/bash

ip tunnel add tun1 mode gre local 10.10.10.1 remote  20.20.20.100 ttl 255

ip link set tun1 up

ip addr add 10.5.5.1/30 /dev tun1

/etc/gre.up

#R-WF

#!/bin/bash

ip tunnel add tun1 mode gre local 20.20.20.100 remote 10.10.10.1 ttl 255

ip link set tun1 up

ip addr add 10.5.5.2/30 /dev tun1


/etc/crontab


@reboot  root  /etc/gre.up
