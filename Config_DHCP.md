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




```
