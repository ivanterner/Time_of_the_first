Компилируем и копируем на хост бинарники в /usr/local/bin/:
postfix_exporter
prometheus
promtool


Создаем сервис для Прометея.
```yaml
cat /etc/systemd/system/prometheus.service

[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
Environment="GOMAXPROCS=1" 
User=root
Group=root
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/local/bin/prometheus \
  --web.config.file=/etc/prometheus/web.yml \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
```

Создаем сервис для Postfix_eprorter.
```yaml
cat /etc/systemd/system/postfix_exporter.service
[Unit]
Description=Prometheus
Documentation=https://github.com/kumina/postfix_exporter
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=root
Group=root
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/postfix_exporter \
  --web.listen-address=127.0.0.1:9154 \
  --web.telemetry-path=/metrics \
  --postfix.logfile_path=/var/log/maillog \

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
```
Перечитываем systemd
```bash
systemctl daemon-reload
```

Создаем конфиг для Прометея /etc/prometheus/prometheus.yml
```yaml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml" 
  # - "second_rules.yml" 

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']

  - job_name: postfix
    static_configs:
      - targets: ['localhost:9154']
```
Создаем конфиг файл авторизации /etc/prometheus/web.yml
```yaml
basic_auth_users:
       admin: 'Твой пароль зашифрованый библиотекой bcrypt'
```
Стартуем сервисы.
```bash
systemctl start prometheus
systemctl start postfix_exporter
```

Добавляем в автозагрузку.
```bash
systemctl enable prometheus
systemctl enable postfix_exporter
```

Для BIND.
```yaml
[Unit]
Description=Prometheus
Documentation=https://github.com/digitalocean/bind_exporter
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/bind_exporter \
  --bind.pid-file=/var/run/named/named.pid \
  --bind.timeout=20s \
  --web.listen-address=0.0.0.0:9153 \
  --web.telemetry-path=/metrics \
  --bind.stats-url=http://localhost:8053/ \
  --bind.stats-groups=server,view,tasks

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
```
