
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

```
