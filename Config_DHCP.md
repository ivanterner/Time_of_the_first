![Карта сети ](/dhcp.png)

Шаг 1: Установка ISC DHCP сервера\
На сервере установите пакет ISC DHCP:
```bash
yum install -y dhcp-server.x86_64
```


Шаг 2: Настройка конфигурационных файлов\
Откройте файл конфигурации /etc/dhcp/dhcpd.conf для редактирования:\
Добавьте или измените следующие строки:
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
Шаг 3: Настройка службы DHCP
Настройте, чтобы служба DHCP использовала правильный интерфейс.
Откройте файл vi /etc/sysconfig/dhcpd и укажите нужный интерфейс:
Измените строку:
```yaml
INTERFACESv4="eth0"
```
Шаг 4: запуск службы DHCP\
После настройки конфигурационных файлов запустите службы DHCP на сервере:
```bash
sudo  systemctl start dhcpd
sudo  systemctl enable dhcpd
```
Шаг 5: Проверка статуса службы\
Убедитесь, что службы DHCP работают корректно:
```bash
sudo systemctl status dhcpd
```


Шаг 6: Установка сервера пересылки запросов.
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




