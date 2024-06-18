Установить mamonsu.
```console
sudo yum install mamonsu-3.5.5-1.el7.noarch
```

Создать БД для мониторинга.
```console
CREATE USER mamonsu WITH PASSWORD 'yourpassword';
GRANT pg_monitor TO mamonsu;
CREATE DATABASE mamonsu OWNER mamonsu;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_ls_dir(text) TO mamonsu;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_stat_file(text) TO mamonsu;
```
Добавить в pg_hba.conf.
```console
host    all             mamonsu     127.0.0.1/32            md5
```

Добавить в агент /etc/mamonsu/agent.conf.
```console
[postgres]
enabled = True
user = mamonsu
password = yourpassword
database = mamonsu
host = localhost
port = 5432
application_name = mamonsu
query_timeout = 10

[zabbix]
enabled = True
client = FQDN
address = 192.168.100.199 #Адресс сервера zabbix
port = 10051
re_send = False
```

Подготовить БД.
```console
mamonsu bootstrap -M mamonsu -x -c /etc/mamonsu/agent.conf -d mamonsu -U postgres --host 127.0.0.1 --port 5432
```

Запустить сервис и добавить в автозагрузку.
```console
sudo systemctl start mamonsu.service
sudo systemctl enable mamonsu.service
```

Команда для просмотра метрик в консоли.
```console
sudo mamonsu agent metric-list
```
Подключить библиотеки в файле postgresql.conf или через ppem.
```console
shared_preload_libraries = 'online_analyze, plantuner, pg_stat_statements, pgpro_stats'
```
Подключить шаблон в zabbix.
