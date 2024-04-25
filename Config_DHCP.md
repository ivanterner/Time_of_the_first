![Карта сети ](/dhcp.png)
```console
#Установка dcp сервера
yum install dhcp-server.x86_64

#Копирем конфиг
cp /usr/share/doc/dhcp-server/dhcpd.conf.example /etc/dhcp/dhcpd.conf

#Настраиваем сервер
vim dhcpd.conf

#Данными строчками задается доменное имя и DNS-сервер:
option domain-name "skill39.wsr";
option domain-name-servers 172.16.20.10;

#Данные строки настраивают время аренды IP-адреса:
default-lease-time 600;
max-lease-time 7200;

#Настройка опции динамического обновления DNS:
ddns-update-style interim;
ddns-updates on;

log-facility local7;

subnet 172.16.20.0 netmask 255.255.255.0 {
}
subnet 172.16.50.0 netmask 255.255.255.0 {
}

#Данными строками указываются подсети, которые будет обслуживать DNS. В их
#настройке указывается диапазон адресов с ключевым словом range и список IP-адресов
#маршрутизаторов для клиентской сети ключевым словом option routers:
subnet 172.16.100.0 netmask 255.255.255.0 {
  range 172.16.100.65 172.16.100.75;
  option routers 172.16.100.1;
}

subnet 172.16.200.0 netmask 255.255.255.0 {
  range 172.16.200.65 172.16.200.75;
  option routers 172.16.200.1;
}

host l-cli-b {
  hardware ethernet 08:00:07:26:c0:a5;
  fixed-address 172.16.200.61;
}

#Настраваем запуск на интефейсах 
vim /etc/default/isc-dhcp-server
INTERFACESv4="enp1s0 enp7s0"

#Запускаем демон
systemctl start dhcpd.service
systemctl enable dhcpd.service 

#Установка сервера пересылки
yum install dhcp-relay.x86_64
#Настраивается через systemd
vim /etc/systemd/system/multi-user.target.wants/dhcrelay.service
[Unit]
Description=DHCP Relay Agent Daemon
Documentation=man:dhcrelay(8)
Wants=network-online.target
After=network-online.target

[Service]
Type=notify
ExecStart=/usr/sbin/dhcrelay -d --no-pid  172.16.50.2 -i enp1s0 enp7s0
StandardError=null

[Install]
WantedBy=multi-user.target

#Запускаем демон
systemctl enable dhcrelay.service 
systemctl start dhcrelay.service 
systemctl status dhcrelay.service 






5.

User
1. Установите Docker и Docker Compose.
2. Создайте локальный Docker Registry.
3. В домашней директории хоста создайте файл name.txt и запишите в него строку experts.
4. Напишите Dockerfile для приложения HelloFIRPO.
1. В качестве базового образа используйте alpine
2. Сделайте рабочей директорию /hello и скопируйте в неё name.txt
3. Контейнер при запуске должен выполнять команду echo, которая выводит сообщение "Hello, FIRPO! Greetings from" и затем содержимое файла name.txt, после чего завершать свою работу.
ChatGPT
Вот Dockerfile для вашего приложения HelloFIRPO:

Dockerfile
Copy code
# Используем alpine как базовый образ
FROM alpine

# Устанавливаем необходимые пакеты (можно пропустить, если они не нужны)
RUN apk update && apk add --no-cache \
    bash \
    && rm -rf /var/cache/apk/*

# Устанавливаем рабочую директорию внутри контейнера
WORKDIR /hello

# Копируем файл name.txt из хоста в контейнер
COPY name.txt .

# Команда, которую выполнит контейнер при запуске
CMD echo "Hello, FIRPO! Greetings from $(cat name.txt)"
Теперь, чтобы собрать образ и запустить контейнер, следуйте этим шагам:

Сохраните этот Dockerfile в файл с именем "Dockerfile" в вашей рабочей директории.
Поместите файл name.txt в вашу домашнюю директорию на хосте.
Соберите Docker-образ с помощью команды docker build:
Copy code
docker build -t hellofirpo .
Здесь "hellofirpo" - имя образа, которое вы можете выбрать по своему усмотрению.
Запустите контейнер с помощью команды docker run:
arduino
Copy code
docker run --rm hellofirpo
Опция --rm удаляет контейнер после его завершения, чтобы избежать накопления неиспользуемых контейнеров.
Теперь при запуске контейнера вы увидите вывод "Hello, FIRPO! Greetings from experts".



ChatGPT 3.5

User
1. Установите Docker и Docker Compose.
2. Создайте локальный Docker Registry.
3. В домашней директории хоста создайте файл name.txt и запишите в него строку experts.
4. Напишите Dockerfile для приложения HelloFIRPO.
1. В качестве базового образа используйте alpine
2. Сделайте рабочей директорию /hello и скопируйте в неё name.txt
3. Контейнер при запуске должен выполнять команду echo, которая выводит сообщение "Hello, FIRPO! Greetings from" и затем содержимое файла name.txt, после чего завершать свою работу.
ChatGPT
Вот Dockerfile для вашего приложения HelloFIRPO:

Dockerfile
Copy code
# Используем alpine как базовый образ
FROM alpine

# Устанавливаем необходимые пакеты (можно пропустить, если они не нужны)
RUN apk update && apk add --no-cache \
    bash \
    && rm -rf /var/cache/apk/*

# Устанавливаем рабочую директорию внутри контейнера
WORKDIR /hello

# Копируем файл name.txt из хоста в контейнер
COPY name.txt .

# Команда, которую выполнит контейнер при запуске
CMD echo "Hello, FIRPO! Greetings from $(cat name.txt)"
Теперь, чтобы собрать образ и запустить контейнер, следуйте этим шагам:

Сохраните этот Dockerfile в файл с именем "Dockerfile" в вашей рабочей директории.
Поместите файл name.txt в вашу домашнюю директорию на хосте.
Соберите Docker-образ с помощью команды docker build:
Copy code
docker build -t hellofirpo .
Здесь "hellofirpo" - имя образа, которое вы можете выбрать по своему усмотрению.
Запустите контейнер с помощью команды docker run:
arduino
Copy code
docker run --rm hellofirpo
Опция --rm удаляет контейнер после его завершения, чтобы избежать накопления неиспользуемых контейнеров.
Теперь при запуске контейнера вы увидите вывод "Hello, FIRPO! Greetings from experts".

6.
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
ChatGPT
Давайте начнем с шага 5, сборки образа приложения App и загрузки его в ваш локальный Registry. После этого мы перейдем к шагу 6 и созданию стека контейнеров с приложением MediaWiki и базой данных.

Шаг 5: Сборка образа приложения App и загрузка в локальный Registry
Для этого нам понадобится Dockerfile для приложения App и несколько команд Docker для сборки и загрузки образа. Предполагая, что у вас уже есть Dockerfile для приложения App, и его версия 1.0, давайте выполним эти шаги:

Перейдите в директорию с Dockerfile приложения App.
Убедитесь, что ваш Docker Registry запущен и доступен на вашем локальном хосте. Если вы только что установили его, он должен быть доступен по адресу localhost:5000.
Соберите образ приложения App с помощью команды docker build:
bash
Copy code
docker build -t localhost:5000/app:1.0 .
Это соберет образ с тегом 1.0 и пометит его для вашего локального Registry.
Загрузите собранный образ в ваш локальный Registry с помощью команды docker push:
bash
Copy code
docker push localhost:5000/app:1.0
Теперь ваш образ приложения App доступен в вашем локальном Docker Registry.

Шаг 6: Создание стека контейнеров с приложением MediaWiki и базой данных
Для этого нам понадобится файл docker-compose.yml, который опишет наш стек контейнеров.

Вот пример docker-compose.yml для MediaWiki и базы данных MySQL:

yaml
Copy code
version: '3'

services:
  wiki:
    image: mediawiki
    ports:
      - "80:80"
    volumes:
      - ~/redos/LocalSettings.php:/var/www/html/LocalSettings.php
    networks:
      - appnet

  db:
    image: mysql
    environment:
      MYSQL_DATABASE: mediawiki
      MYSQL_USER: wiki
      MYSQL_PASSWORD: P@ssw0rd
      MYSQL_ROOT_PASSWORD: rootpassword
    volumes:
      - dbvolume:/var/lib/mysql
    networks:
      - appnet

volumes:
  dbvolume:

networks:
  appnet:
    external: true
Обратите внимание, что я использовал директиву external для сети appnet, что предполагает, что у вас уже есть созданная сеть с этим именем. Если у вас нет такой сети, вы можете создать ее с помощью docker network create appnet.

Теперь, когда у вас есть docker-compose.yml, вы можете запустить стек контейнеров с помощью команды:

Copy code
docker-compose up -d
Это запустит стек контейнеров в фоновом режиме. После этого MediaWiki будет доступна извне через порт 80
```
