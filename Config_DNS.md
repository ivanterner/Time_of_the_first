
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

#Проверям конфиг, стратуем службу.
named-checkconf /etc/named.conf 
systemctl start named
systemctl enable named




Ansible

---
- name: Gather IP addresses and hostnames
  hosts: all
  gather_facts: yes

  tasks:
    - name: Gather IP addresses and hostnames
      debug:
        msg: "{{ inventory_hostname }} - {{ ansible_default_ipv4.address }}"
      register: host_info

- name: Save output to file
  hosts: localhost
  tasks:
    - name: Save output to file
      copy:
        content: "{{ host_info.results | map(attribute='msg') | join('\n') }}"
        dest: /etc/ansible/output.yaml


LVM



Вот инструкции по настройке дисковой подсистемы на серверах SRV-HQ и SRV-BR:

Настройка зеркалируемого LVM тома на SRV-HQ:
Подготовка дисков:
Убедитесь, что у вас есть два неразмеченных жестких диска.
Создайте физический том (PV) на каждом диске: pvcreate /dev/sdX (где /dev/sdX - ваш жесткий диск).
Создайте группу томов (VG) на основе этих физических томов: vgcreate vg_mirror /dev/sdX /dev/sdY.
Создание зеркалируемого LVM тома:
Создайте логический том (LV) с зеркалированием: lvcreate -m1 -n lv_mirror -l 100%FREE vg_mirror.
Форматируйте новый том: mkfs.ext4 /dev/vg_mirror/lv_mirror.
Настройка автоматического монтирования:
Добавьте запись в файл /etc/fstab для автоматического монтирования: /dev/vg_mirror/lv_mirror /opt/data ext4 defaults 0 2.
Настройка stripped LVM тома с шифрованием на SRV-BR:
Подготовка дисков:
Убедитесь, что у вас есть два неразмеченных жестких диска.
Создайте физический том (PV) на каждом диске: pvcreate /dev/sdX.
Создайте группу томов (VG) на основе этих физических томов: vgcreate vg_stripped /dev/sdX /dev/sdY.
Создание stripped LVM тома:
Создайте логический том (LV) с распределением данных на несколько дисков: lvcreate -i2 -I64 -L 100%FREE -n lv_stripped vg_stripped.
Зашифруйте созданный том: cryptsetup luksFormat /dev/vg_stripped/lv_stripped.
Откройте зашифрованный том: cryptsetup luksOpen /dev/vg_stripped/lv_stripped lv_stripped.
Создайте файловую систему: mkfs.ext4 /dev/mapper/lv_stripped.
Настройка автоматического монтирования:
Добавьте запись в файл /etc/crypttab для автоматического открытия зашифрованного тома: lv_stripped UUID=<UUID_зашифрованного_тома> none luks.
Добавьте запись в файл /etc/fstab для автоматического монтирования: /dev/mapper/lv_stripped /opt/data ext4 defaults 0 2.
Пожалуйста, не забудьте заменить /dev/sdX на соответствующие устройства вашего сервера и <UUID_зашифрованного_тома> на UUID вашего зашифрованного тома.







# Используем alpine в качестве базового образа
FROM alpine

# Устанавливаем рабочую директорию
WORKDIR /hello

# Копируем файл name.txt в рабочую директорию контейнера
COPY name.txt .

# При запуске контейнер выполняет команду echo для вывода приветственного сообщения и содержимого файла name.txt
CMD ["sh", "-c", "echo 'Hello, FIRPO! Greetings from' && cat name.txt"]





Давайте создадим файл wiki.yml для настройки приложения MediaWiki с использованием Docker Compose, учитывая указанные требования:version: '3.8'

services:
  # Сервис MediaWiki
  wiki:
    image: mediawiki
    container_name: wiki
    restart: always
    ports:
      - "80:80"
    volumes:
      # Монтируем файл LocalSettings.php из домашней папки пользователя redos
      - /home/redos/LocalSettings.php:/var/www/html/LocalSettings.php
    networks:
      - app_net

  # Сервис базы данных MySQL
  db:
    image: mysql
    container_name: db
    restart: always
    environment:
      MYSQL_DATABASE: mediawiki
      MYSQL_USER: wiki
      MYSQL_PASSWORD: P@ssw0rd
      MYSQL_ROOT_PASSWORD: P@ssw0rd
    volumes:
      # Создаем volume для хранения данных базы
      - dbvolume:/var/lib/mysql
    networks:
      - app_net

volumes:
  # Volume для хранения данных базы
  dbvolume:

networks:
  # Создаем отдельную сеть для связи между приложением и базой данных
  app_net:








ChatGPT 3.5

User
8. Создайте балансировщик нагрузки.
1. Сохраните внешний адрес балансировщика нагрузки в файле
/home/redos/lb.ip.
2. Ограничьте внешний доступ протоколами http и https.
3. Балансировка нагрузки должна использовать алгоритм round robin.
4. При обращении на внешний адрес балансировщика нагрузки должен выводиться ответ от приложения на внутреннем сервере.
2. Виртуальные машины должны соответствовать следующим характеристикам.
1. Операционная система: redos.
2. Количество vCPU: 1.
3. Объём оперативной памяти: 1024 МБ.
4. Объём диска: 5 ГБ.
5. Тип диска: HDD.
6. Разместите виртуальные машины в регионе Москва.
7. Разместите Web1 в зоне доступности MS1.
8. Разместите Web2 в зоне доступности GZ1.
3. На машине ControlVM создайте скрипт cloudinit.sh.
1. В качестве рабочей директории используйте путь /home/redos/bin.
2. Используйте файл /home/redos/bin/cloud.conf для указания настроек для подключения к облачному провайдеру.
1. При выполнении проверки, эксперты могут изменить настройки только в файле cloud.conf. Другие файлы редактироваться не будут.
2. Вы можете оставить любые понятные комментарии в файле cloud.conf.
3. В качестве шаблона файла cloud.conf можно использовать следующую структуру:
#!/usr/bin/env bash#Существующий ID проекта в облакеPROJECT_ID='34359eb284cc44c98b9fac5e435d7a57'#Имя пользователя для подключения к облачному провайдеруUSER_NAME='skill39@moscow.local'#Пароль в открытом виде для подключения к облачному провайдеруPASSWORD='P@ssw0rd2023'
3. Скрипт должен выполняться из любой директории без явного указания пути к исполняемому файлу.
4. Выполнение задания ожидается с использованием инструментов Terraform и/или OpenStack CLI. Однако, вы вправе выбрать другие инструменты, не противоречащие условиям задания и правилам соревнования.

Развертывание приложений в Docker
1. На машине ControlVM.
1. Установите Docker и Docker Compose.
2. Создайте локальный Docker Registry.
3. В домашней директории хоста создайте файл name.txt и запишите в него строку experts.
4. Напишите Dockerfile для приложения HelloFIRPO.
1. В качестве базового образа используйте alpine
2. Сделайте рабочей директорию /hello и скопируйте в неё name.txt
3. Контейнер при запуске должен выполнять команду echo, которая выводит сообщение "Hello, FIRPO! Greetings from" и затем содержимое файла name.txt, после чего завершать свою работу.
5. Соберите образ приложения App и загрузите его в ваш Registry.
1. Используйте номер версии 1.0 для вашего приложения
2. Образ должен быть доступен для скачивания и дальнейшего запуска на локальной машине.
6. Создайте в домашней директории пользователя redos файл wiki.yml для приложения MediaWiki.
1. Средствами docker compose должен создаваться стек контейнеров с приложением MediaWiki и базой данных.
1. Используйте два сервиса
2. Основной контейнер MediaWiki должен называться wiki и использовать образ mediawiki
3. Файл LocalSettings.php с корректными настройками должен находиться
в домашней папке пользователя redos и автоматически
монтироваться в образ.
4. Контейнер с базой данных должен называться db и использовать образ mysql
5. Он должен создавать базу с названием mediawiki, доступную по стандартному порту, для пользователя wiki с паролем P@ssw0rd
6. База должна храниться в отдельном volume с названием dbvolume
7. База данных должна находиться в одной сети с приложением App2, но не должна быть доступна снаружи.
8. MediaWiki должна быть доступна извне через порт 80.
7. Настройте мониторинг с помощью NodeExporter, Prometheus и Grafana.
1. Создайте в домашней директории пользователя ubuntu файл monitoring.yml для Docker Compose
1. Используйте контейнеры NodeExporter, Prometheus и Grafana для сбора, обработки и отображения метрик.
2. Настройте Dashboard в Grafana, в котором будет отображаться загрузка CPU, объём свободной оперативной памяти и места на диске.
3. Интерфейс Grafana должен быть доступен по внешнему адресу на порту 3000.
Развёртывания облачных сервисов
1. На машине ControlVM создайте скрипт /home/redos/bin/DeployApp.sh.
1. Скрипт должен выполняться из любой директории без явного указания пути кисполняемому файлу.
2. Подготовьте web-приложение App1
1. Скачайте файлы app1.py и Dockerfile по адресу:
https://github.com/auteam-usr/moscow39
2. Соберите образ приложения и загрузите его в любой репозиторий Docker на ваше усмотрение.
3. Команда DeployApp.sh должна запускать средства автоматизации для настройки
операционных систем.
1. Разверните web-приложение App2 из репозитория Docker на виртуальных
машинах Web1 и Web2.
2. Обеспечьте балансировку нагрузки между Web1 и Web2.
3. Обеспечьте внешний доступ к web-приложению по протоколу https.
4. При обращении по протоколу http должно выполняться автоматическое
перенаправления на протокол https.
5. Обеспечивать доверие сертификату не требуется.
ChatGPT
Для решения этой задачи требуется выполнить ряд шагов:

Настройка виртуальных машин:
Создать две виртуальные машины (Web1 и Web2) в регионе Москва с указанными характеристиками.
Разместить Web1 в зоне доступности MS1, а Web2 в зоне доступности GZ1.
Настройка балансировщика нагрузки:
Создать балансировщик нагрузки с внешним доступом по протоколам HTTP и HTTPS, используя алгоритм round robin.
Сохранить внешний адрес балансировщика в файле /home/redos/lb.ip.
Настройка Docker и Docker Compose:
Установить Docker и Docker Compose на машину ControlVM.
Создать локальный Docker Registry.
Написать Dockerfile для приложения HelloFIRPO и собрать образ приложения с номером версии 1.0.
Создать файл name.txt с содержимым "experts" в домашней директории хоста.
Создать файл wiki.yml для приложения MediaWiki, используя Docker Compose.
Настроить контейнеры MediaWiki и базы данных в wiki.yml.
Настройка мониторинга с помощью NodeExporter, Prometheus и Grafana:
Создать файл monitoring.yml для Docker Compose, включающий контейнеры NodeExporter, Prometheus и Grafana.
Настроить Grafana Dashboard для отображения метрик CPU, оперативной памяти и дискового пространства.
Обеспечить доступ к интерфейсу Grafana по внешнему адресу на порту 3000.
Развёртывание облачных сервисов:
Создать скрипт /home/redos/bin/DeployApp.sh для развертывания web-приложения App1 и App2.
Подготовить и загрузить образ приложения App1 в Docker репозиторий.
Развернуть web-приложение App2 на виртуальных машинах Web1 и Web2 с балансировкой нагрузки.
Обеспечить внешний доступ к web-приложению по протоколу HTTPS с автоматическим перенаправлением с HTTP.
Для выполнения каждого из этих шагов можно использовать инструменты Terraform, OpenStack CLI, Docker, Docker Compose и другие инструменты, если они соответствуют условиям задачи и правилам соревнования.

User
Напишите Dockerfile для приложения HelloFIRPO.
1. В качестве базового образа используйте alpine
2. Сделайте рабочей директорию /hello и скопируйте в неё name.txt
3. Контейнер при запуске должен выполнять команду echo, которая выводит сообщение "Hello, FIRPO! Greetings from" и затем содержимое файла name.txt, после чего завершать свою работу.
ChatGPT
Вот Dockerfile для приложения HelloFIRPO, учитывающий указанные требования:

Dockerfile
Copy code
# Используем alpine в качестве базового образа
FROM alpine

# Устанавливаем рабочую директорию
WORKDIR /hello

# Копируем файл name.txt в рабочую директорию контейнера
COPY name.txt .

# При запуске контейнер выполняет команду echo для вывода приветственного сообщения и содержимого файла name.txt
CMD ["sh", "-c", "echo 'Hello, FIRPO! Greetings from' && cat name.txt"]
Этот Dockerfile создаст образ, который при запуске будет выводить сообщение "Hello, FIRPO! Greetings from" и затем содержимое файла name.txt, который скопирован внутрь контейнера.




User
1. Используйте номер версии 1.0 для вашего приложения
2. Образ должен быть доступен для скачивания и дальнейшего запуска на локальной машине.
6. Создайте в домашней директории пользователя redos файл wiki.yml для приложения MediaWiki.
1. Средствами docker compose должен создаваться стек контейнеров с приложением MediaWiki и базой данных.
1. Используйте два сервиса
2. Основной контейнер MediaWiki должен называться wiki и использовать образ mediawiki
3. Файл LocalSettings.php с корректными настройками должен находиться
в домашней папке пользователя redos и автоматически
монтироваться в образ.
4. Контейнер с базой данных должен называться db и использовать образ mysql
5. Он должен создавать базу с названием mediawiki, доступную по стандартному порту, для пользователя wiki с паролем P@ssw0rd
6. База должна храниться в отдельном volume с названием dbvolume
7. База данных должна находиться в одной сети с приложением App2, но не должна быть доступна снаружи.
8. MediaWiki должна быть доступна извне через порт 80.
ChatGPT
Давайте создадим файл wiki.yml для настройки приложения MediaWiki с использованием Docker Compose, учитывая указанные требования:

yaml
Copy code
version: '3.8'

services:
  # Сервис MediaWiki
  wiki:
    image: mediawiki
    container_name: wiki
    restart: always
    ports:
      - "80:80"
    volumes:
      # Монтируем файл LocalSettings.php из домашней папки пользователя redos
      - /home/redos/LocalSettings.php:/var/www/html/LocalSettings.php
    networks:
      - app_net

  # Сервис базы данных MySQL
  db:
    image: mysql
    container_name: db
    restart: always
    environment:
      MYSQL_DATABASE: mediawiki
      MYSQL_USER: wiki
      MYSQL_PASSWORD: P@ssw0rd
      MYSQL_ROOT_PASSWORD: P@ssw0rd
    volumes:
      # Создаем volume для хранения данных базы
      - dbvolume:/var/lib/mysql
    networks:
      - app_net

volumes:
  # Volume для хранения данных базы
  dbvolume:

networks:
  # Создаем отдельную сеть для связи между приложением и базой данных
  app_net:
Этот файл docker-compose.yml создаст стек контейнеров с двумя сервисами: MediaWiki и MySQL (для базы данных). MediaWiki будет доступен извне через порт 80, а база данных MySQL будет доступна только для приложения App2 через отдельную сеть app_net. Deв файла LocalSettings.php, который требуется для настройки MediaWiki, должен быть предварительно размещен в домашней папке пользователя redos.

User
7. Настройте мониторинг с помощью NodeExporter, Prometheus и 
Давайте создадим файл monitoring.yml для настройки мониторинга с помощью Docker Compose с использованием NodeExporter, Prometheus и Grafana. Затем мы настроим Dashboard в Grafana для отображения метрик.

yaml
Copy code
version: '3.8'

services:
  # Сервис NodeExporter для сбора метрик с хостовой системы
  node-exporter:
    image: prom/node-exporter
    container_name: node-exporter
    restart: always
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)'

  # Сервис Prometheus для сбора и хранения метрик
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    restart: always
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  # Сервис Grafana для отображения метрик в виде Dashboard
  grafana:
    image: grafana/grafana
    container_name: grafana
    restart: always
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=your_password
      - GF_SECURITY_ADMIN_USER=admin
    volumes:
      - grafana_data:/var/lib/grafana

volumes:
  grafana_data:
В файле prometheus.yml, который должен быть в той же директории, что и monitoring.yml, настроим Prometheus для сбора метрик от NodeExporter:

yaml
Copy code
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
После запуска Docker Compose с этим файлом конфигурации, Prometheus будет собирать метрики от NodeExporter, а Grafana будет доступна по внешнему адресу на порту 3000. В Grafana мы можем создать Dashboard и добавить панели для отображения загрузки CPU, объема свободной оперативной памяти и места на диске, используя соответствующие запросы Prometheus.

```
