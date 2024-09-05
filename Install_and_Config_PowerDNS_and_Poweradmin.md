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
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] Y
Enabled successfully!
Reloading privilege tables..
 ... Success!


You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] n
 ... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
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
