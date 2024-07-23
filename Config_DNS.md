Устанавливаем bind.
```bash
yum install -y bind
```
Редактируем файл vim /etc/named.conf.

```code
options {
	listen-on port 53 { any; }; #Слушаем на всех интерфейсах 
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	secroots-file	"/var/named/data/named.secroots";
	recursing-file	"/var/named/data/named.recursing";
	allow-query     { any; }; #Отвечаем на запросы любым клиентам
	forwarders { 10.10.10.10; }; #Сервер условной пересылки 
  recursion yes; #Включение рекурсии
  dnssec-validation no; #Отключение dnssec

	managed-keys-directory "/var/named/dynamic";
	geoip-directory "/usr/share/GeoIP";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";

	/* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
	include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "redos.lab." IN {
	type master;
	file "/var/named/redos.lab";
};
#Прямая зона
zone "100.168.192.in-addr.arpa" IN {
        type master;
        file "/var/named/100.168.192.in-addr.arpa";
};
#Обратная зона
zone "." IN {
	type hint;
	file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```
Создаем файл зоны redos.lab.
```yaml
$TTL 1D
@		IN SOA	redos.lab. redos-01 (
					101	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
	 	IN	NS	redos-01.redos.lab.
@		IN	A	192.168.100.6	
redos-01 	IN	A	192.168.100.6
redos-02	IN	A	192.168.100.7
www		IN	A	192.168.100.6
		IN	MX 10	mail
mail          	IN      A      192.168.100.6
```
Создаем файл зоны 100.168.192.in-addr.arpa.
```yaml
$TTL 1D
@		IN SOA	redos.lab. redos-01.redos.lab. (
					102	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
	 	IN	NS	redos-01.redos.lab.
6	 	IN	PTR	redos-01.redos.lab.
7		IN	PTR	redos-02.redos.lab.
```
Проверям конфиг.
```bash
named-checkconf /etc/named.conf 
```
Проверяем файлы зон.
```bash
named-checkzone 100.168.192.in-addr.arpa 100.168.192.in-addr.arpa
named-checkzone redos.lab. redos.lab
``` 
Стратуем службу.
```bash
systemctl start named
systemctl enable named
```
