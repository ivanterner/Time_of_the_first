Устанавливаем bind
```console
yum install bind
```
Редактируем файл vim /etc/named.conf


Слушаем на всех интрфейсач listen-on port 53 { any; };

Отвечаем на запросы любым клиентам allow-query     { any; };

Сервер уловной пресылки DNS forwarders { 10.10.10.10; };

Включение рекурсии recursion yes;

Отключение dnssec-validation no;;

Прямая зона zone "redos.lab."

Обратная зона zone "100.168.192.in-addr.arpa"
```console
options {
	listen-on port 53 { any; };
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	secroots-file	"/var/named/data/named.secroots";
	recursing-file	"/var/named/data/named.recursing";
	allow-query     { any; };
	forwarders { 10.10.10.10; };
  recursion yes;
  dnssec-validation no;

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

zone "100.168.192.in-addr.arpa" IN {
        type master;
        file "/var/named/100.168.192.in-addr.arpa";
};

zone "." IN {
	type hint;
	file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

#Создаем файл зоны redos.lab
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


#Создаем файл зоны 100.168.192.in-addr.arpa
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
```console
named-checkconf /etc/named.conf 
```
Проверяем файлы зон.
```console
named-checkzone 100.168.192.in-addr.arpa 100.168.192.in-addr.arpa
named-checkzone redos.lab. redos.lab
``` 
Стратуем службу.
```console
systemctl start named
systemctl enable named
```
