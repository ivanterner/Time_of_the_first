
[Manual]\
["https://www.howtoforge.com/how-to-install-powerdns-and-poweradmin-on-centos-7/?__cf_chl_tk=5JdDWkPy6sjQrWNFZ5aquCYsWCHIFaoMsPrQTj9L_oA-1725534826-0.0.1.1-5054"]

Устанавливаем MariaDB.
```bash
yum -y install mariadb mariadb-server
```
Стратуем службу mariadb.
```bash
systemctl start mariadb
systemctl enable mariadb
```

Запускаем скрипт настройки mysql_secure.
```bash
mysql_secure_installation
```

```yaml
Switch to unix_socket authentication [Y/n] Y
Change the root password? [Y/n] Y
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```

Заходим в mysql.
```bash
mysql -u root -p
```
Создаем БД powerdns.
```sql
create database powerdns;
grant all privileges on powerdns.* to pdns@localhost identified by 'твой пароль';
flush privileges;
```
Переходим в БД.
```sql
use powerdns;
```
Создем таблицы.
```sql
CREATE TABLE domains (
 id INT AUTO_INCREMENT,
 name VARCHAR(255) NOT NULL,
 master VARCHAR(128) DEFAULT NULL,
 last_check INT DEFAULT NULL,
 type VARCHAR(6) NOT NULL,
 notified_serial INT DEFAULT NULL,
 account VARCHAR(40) DEFAULT NULL,
 PRIMARY KEY (id)
 ) Engine=InnoDB;
 
 CREATE UNIQUE INDEX name_index ON domains(name);
 
 
 CREATE TABLE records (
 id BIGINT AUTO_INCREMENT,
 domain_id INT DEFAULT NULL,
 name VARCHAR(255) DEFAULT NULL,
 type VARCHAR(10) DEFAULT NULL,
 content VARCHAR(64000) DEFAULT NULL,
 ttl INT DEFAULT NULL,
 prio INT DEFAULT NULL,
 change_date INT DEFAULT NULL,
 disabled TINYINT(1) DEFAULT 0,
 ordername VARCHAR(255) BINARY DEFAULT NULL,
 auth TINYINT(1) DEFAULT 1,
 PRIMARY KEY (id)
 ) Engine=InnoDB;
 
 CREATE INDEX nametype_index ON records(name,type);
 CREATE INDEX domain_id ON records(domain_id);
 CREATE INDEX recordorder ON records (domain_id, ordername);
 
 
 CREATE TABLE supermasters (
 ip VARCHAR(64) NOT NULL,
 nameserver VARCHAR(255) NOT NULL,
 account VARCHAR(40) NOT NULL,
 PRIMARY KEY (ip, nameserver)
 ) Engine=InnoDB;
 
 
 CREATE TABLE comments (
 id INT AUTO_INCREMENT,
 domain_id INT NOT NULL,
 name VARCHAR(255) NOT NULL,
 type VARCHAR(10) NOT NULL,
 modified_at INT NOT NULL,
 account VARCHAR(40) NOT NULL,
 comment VARCHAR(64000) NOT NULL,
 PRIMARY KEY (id)
 ) Engine=InnoDB;
 
 CREATE INDEX comments_domain_id_idx ON comments (domain_id);
 CREATE INDEX comments_name_type_idx ON comments (name, type);
 CREATE INDEX comments_order_idx ON comments (domain_id, modified_at);
 
 
 CREATE TABLE domainmetadata (
 id INT AUTO_INCREMENT,
 domain_id INT NOT NULL,
 kind VARCHAR(32),
 content TEXT,
 PRIMARY KEY (id)
 ) Engine=InnoDB;
 
 CREATE INDEX domainmetadata_idx ON domainmetadata (domain_id, kind);
 
 
 CREATE TABLE cryptokeys (
 id INT AUTO_INCREMENT,
 domain_id INT NOT NULL,
 flags INT NOT NULL,
 active BOOL,
 content TEXT,
 PRIMARY KEY(id)
 ) Engine=InnoDB;
 
 CREATE INDEX domainidindex ON cryptokeys(domain_id);
 
 
 CREATE TABLE tsigkeys (
 id INT AUTO_INCREMENT,
 name VARCHAR(255),
 algorithm VARCHAR(50),
 secret VARCHAR(255),
 PRIMARY KEY (id)
 ) Engine=InnoDB;
 
 CREATE UNIQUE INDEX namealgoindex ON tsigkeys(name, algorithm);
```
Выходи из mysql.
```sql
quit;
```
Устанавливаем PoweDNS.
```bash
yum -y install pdns pdns-backend-mysql bind-utils
```
Создаем резервную копию кофига.
```bash
cd /etc/pdns/
cp pdns.conf pdns.conf.back
```
Правим конфиг.
```yaml
#launch=bind
launch=gmysql
gmysql-host=localhost
gmysql-user=pdns
gmysql-password=твой пароль
gmysql-dbname=powerdns
security-poll-suffix=
setgid=pdns
setuid=pdns
```
Стратуем службу pdns.
```bash
systemctl start pdns
systemctl enable pdns
```
Проверяем порт.
```bash
ss -nlput | grep :53
```
```bash
udp   UNCONN 0      0           0.0.0.0:53        0.0.0.0:*    users:(("pdns_server",pid=4124,fd=5))   
udp   UNCONN 0      0           [::]:53           [::]:*    users:(("pdns_server",pid=4124,fd=6))   
tcp   LISTEN 0      128         0.0.0.0:53        0.0.0.0:*    users:(("pdns_server",pid=4124,fd=7))   
tcp   LISTEN 0      128         [::]:53           [::]:*    users:(("pdns_server",pid=4124,fd=8)
```
Устанавливаем Apache и PHP.
```bash
yum -y install httpd php php-devel php-gd php-imap php-ldap php-mysql
php-odbc php-pear php-xml php-xmlrpc php-mbstring php-mcrypt php-mhash gettext
```
Устанавливаем PHP-pear.
```bash
php-pear-MDB2.noarch
```
Стратуем службу httpd.
```bash
systemctl start httpd
systemctl enable httpd
```
Переходи в директорию веб сервера и скачиваем исходник.
```bash
cd /var/www/html/
 wget http://downloads.sourceforge.net/project/poweradmin/poweradmin-2.1.7.tgz
```
Распакуем и переименуем.
```bash
tar -xzf poweradmin-2.1.7.tgz
poweradmin-2.1.7  poweradmin-2.1.7.tgz
```
Переходим в вебку и настраиваем.
http://192.168.100.4//poweradmin/install/
![PDNS](/img/pdns1.png)
![PDNS](/img/pdns2.png)
![PDNS](/img/pdns3.png)
![PDNS](/img/pdns4.png)
На этом шаге нас просят вернутся в mysql и создать пользователя.
![PDNS](/img/pdns5.png)

Идем в mysql и создаем пользователя.
```mysql
GRANT SELECT, INSERT, UPDATE, DELETE
ON powerdns.*
TO 'pdns'@'localhost'
IDENTIFIED BY 'твой пароль';
```
На этом шаге нас просят создать файл /inc/config.inc.php и наполнить его.
![PDNS](/img/pdns5.png)

```yaml
<?php

$db_host = 'localhost';
$db_user = 'pdns';
$db_pass = 'твой пароль';
$db_name = 'powerdns';
$db_type = 'mysql';
$db_layer = 'PDO';

$session_key = 'твой ключ';

$iface_lang = 'en_EN';

$dns_hostmaster = 'pdns.redos.test';
$dns_ns1 = 'pdns.redos.test';
$dns_ns2 = '';
```
![PDNS](/img/pdns7.png)
```bash
cd /var/www/html/poweradmin/
cp install/htaccess.dist .htaccess
rm -rf install/
```
Можно входить в систему.
![PDNS](/img/pdns8.png)
