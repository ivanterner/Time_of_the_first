![Карта сети ](/dhcp.png)

Шаг 1: Установка ISC DHCP сервера\
На сервере установите пакет ISC DHCP:
```bash
yum install -y dhcp-server.x86_64
```

Копирем конфиг.
```bash
cp /usr/share/doc/dhcp-server/dhcpd.conf.example /etc/dhcp/dhcpd.conf
```

Настраиваем сервер.
```console
vim dhcpd.conf
```
```yaml
#Данными строчками задается доменное имя и DNS-сервер:
option domain-name "test.loc";
option domain-name-servers 172.16.20.10;

#Данные строки настраивают время аренды IP-адреса:
default-lease-time 600;
max-lease-time 7200;

#Настройка опции динамического обновления DNS:
ddns-update-style interim;
ddns-updates on;

log-facility local7;

subnet 172.16.20.0 netmask 255.255.255.0 {
}
subnet 172.16.50.0 netmask 255.255.255.0 {
}

#Данными строками указываются подсети, которые будет обслуживать DNS. В их
#настройке указывается диапазон адресов с ключевым словом range и список IP-адресов
#маршрутизаторов для клиентской сети ключевым словом option routers:
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
```

Настраваем запуск на интефейсах. 
```bash
vim /etc/sysconfig/dhcpd 
```
```yaml
INTERFACESv4="enp1s0 enp7s0"
```

Запускаем демон.
```bash
systemctl start dhcpd.service
systemctl enable dhcpd.service 
```

Установка сервера пересылки запросов.
```bash
yum install -y dhcp-relay.x86_64
```
Настраивается через systemd.
```bash
vim /etc/systemd/system/multi-user.target.wants/dhcrelay.service
```
```yaml
[Unit]
Description=DHCP Relay Agent Daemon
Documentation=man:dhcrelay(8)
Wants=network-online.target
After=network-online.target

[Service]
Type=notify
ExecStart=/usr/sbin/dhcrelay -d --no-pid  172.16.50.2 -i enp1s0 enp7s0
StandardError=null

[Install]
WantedBy=multi-user.target
```
Запускаем демон.
```bash
systemctl enable dhcrelay.service 
systemctl start dhcrelay.service 
systemctl status dhcrelay.service 
```




