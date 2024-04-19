yum install frr.x86_64

vim /etc/frr/daemons

zebra=yes
bgpd=no
ospfd=yes
ospf6d=no
ripd=no
ripngd=no
isisd=no
pimd=no
pim6d=no
nhrpd=no
eigrpd=no
sharpd=no
pbrd=no
bfdd=no
fabricd=no
vrrpd=no
pathd=no


systemctl start frr
systemctl enable frr
