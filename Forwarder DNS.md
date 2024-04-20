```console
#Устанавливаем bind
yum install bind
#Редактируем файл vim /etc/named.conf
#Слушаем на всех интрфейсач listen-on port 53 { any; };
#Отвечаем на запросы любым клиентам allow-query     { any; };
#Сервер уловной пресылки DNS forwarders { 10.10.10.10; };
#Включение рекурсии recursion yes;
#Отключение dns-sec
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




named-checkconf /etc/named.conf 
systemctl start named
systemctl enable named
```



