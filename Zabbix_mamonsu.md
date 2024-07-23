Установить mamonsu.
```bash
sudo yum install -y mamonsu
```

Создать БД для мониторинга.
```sql
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
```yaml
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
```bash
mamonsu bootstrap -M mamonsu -x -c /etc/mamonsu/agent.conf -d mamonsu -U postgres --host 127.0.0.1 --port 5432
```

Запустить сервис и добавить в автозагрузку.
```bash
sudo systemctl start mamonsu.service
sudo systemctl enable mamonsu.service
```

Команда для просмотра метрик в консоли.
```bash
sudo mamonsu agent metric-list
```
Подключить библиотеки в файле postgresql.conf или через ppem.
```yaml
shared_preload_libraries = 'online_analyze, plantuner, pg_stat_statements, pgpro_stats'
```
Подключить шаблон в zabbix.
