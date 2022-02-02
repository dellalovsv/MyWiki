---
title: Установка/Перенос 0.59
description: Установка и настройка Abills 0.59 под NGiNX на Debian
published: false
date: 2022-02-02T17:58:38.189Z
tags: abills, abills 0.59, debian, nginx
editor: markdown
dateCreated: 2022-02-02T16:35:41.603Z
---

# Настройка Abills 0.59 под NGiNX
## Подготовка системы
```bash
apt update -yq && apt upgrade -yq
apt install libperl-dev  gcc make
```
## Настройка Abills
В файле **config.pl** изменяем параметры для подключения к БД:
```bash
nano /usr/abills/libexec/config.pl
```
```perl
$conf{dbhost}='localhost';
$conf{dbname}='abills'; 
$conf{dbuser}='abills';
$conf{dbpasswd}='sqlpassword'; 
$conf{ADMIN_MAIL}='info@your.domain'; 
$conf{USERS_MAIL_DOMAIN}="your.domain";
# используется для шифрования паролей администраторов и пользователей.
$conf{secretkey}="test12345678901234567890";
```
### Создание директорий и назначение прав файлам
```bash
mkdir /usr/abills/var/ /usr/abills/var/log /usr/abills/backup
touch /usr/abills/var/log/abills.log
chown -Rf www-data /usr/abills/cgi-bin
chown -Rf www-data /usr/abills/Abills/templates
chown -Rf www-data /usr/abills/backup
chmod 755 /usr/abills/cgi-bin/admin/index.cgi
chmod 755 /usr/abills/cgi-bin/index.cgi
```
### Создание symlinks для gzip и mysqldump
```bash
ln -s /bin/gzip /usr/bin/gzip
ln -s /usr/bin/mysqldump /usr/local/bin/mysqldump
```
### Установка Perl модулей
Для установки модулей нужно запустить скрипт **perldeps.pl**. В 0.59 его нет! Его можно взять из версий 0.7x и выше.
```bash
cd /usr/abills/misc/ && perl perldeps.pl apt-get -batch
```
Для установки **Digest::SHA1** нужно установить **cpanm**. После установить модуль.
```bash
apt-get install -yq cpanminus
```
```bash
cpanm Digest::SHA1
```
### После добавления NAS-сервера в биллинг
Чтобы работал гостевой пул, нужно изменить дерективу **$conf{ACCEL_IPOE_GUEST_POOL}** в файле **config.pl**
```bash
nano /usr/abills/libexec/config.pl
```
```perl
$conf{ACCEL_IPOE_GUEST_POOL}=№ наса в биллинге,№ гостевого пула в биллинге
```
### Cron
Открываем файл **crontab**:
```bash
nano /etc/crontab
```
Дописываем:
```bash
*/5 * * * *	root   /usr/abills/libexec/billd -all
1 0 * * *		root    /usr/abills/libexec/periodic daily
1 1 * * *		root    /usr/abills/libexec/periodic monthly
```
Применяем настройки:
```bash
crontab /etc/crontab
```
## [Настройка MySQL (MariaDB)](https://wiki.delovoyadmin.net/ru/NGiNX/install_debian#%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-mariadb-phpmyadmin)
Статья по настройке **MySQL** по ссылке выше.

Добавляем данные дампа БД в **MySQL бд**:
```bash
mysql -u {USER} -p'{PASSWORD}' -D abills < /path/to/dump/db.sql
```
## Настройка Freeradius
### Установка
```bash
apt install freeradius -y
```
### Настройка
Скачиваем последнюю версию [Abills](https://netix.dl.sourceforge.net/project/abills/abills/0.92/abills-0.92.05.tgz)
- Распаковываем;
- Заходим в распакованую директорию **abills**;

Далее выполняем следующии действия:
```bash
rm /etc/freeradius/3.0/sites-enabled/*
cp misc/freeradius/v3/mods-enabled/perl /etc/freeradius/3.0/mods-enabled/perl
cp misc/freeradius/v3/mods-enabled/sql /etc/freeradius/3.0/mods-enabled/sql
cp misc/freeradius/v3/sites-enabled/abills_default /etc/freeradius/3.0/sites-enabled/abills_default
cp misc/freeradius/v3/users /etc/freeradius/3.0/users
```
> В файле **/etc/freeradius/3.0/sites-enabled/abills_default** нужно закомментировать строку **perl** в **Post-Authentication**.
{.is-warning}

```bash
#  Post-Authentication
post-auth {
#       perl   <<<------ Вот эта строка
        Post-Auth-Type REJECT {
                perl
        }
}
```
Отключаем поддержку **sql** в **freeradius**:
```bash
mv /etc/freeradius/3.0/mods-enabled/sql /etc/freeradius/3.0/mods-available/
```
Добавляем клиента в **freeradius**:
```bash
echo '' > /etc/freeradius/3.0/clients.conf
nano /etc/freeradius/3.0/clients.conf
```
```bash
client IP-ADDRESS {
    ipaddr = IP-ADDRESS
    secret = blablabla
    shortname = IPoE
}
```
## Настройка NGiNX
Установка NGiNX описана в другой статье [здесь](https://wiki.delovoyadmin.net/ru/NGiNX/install_debian#%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-nginx-php).
### Настройка хоста для клиентов/абонентов
```bash
nano /etc/nginx/sites-available/clients.abills
```
```nginx
    listen 80;
    server_name client.billing.net;

    access_log off;
    error_log /var/log/nginx/billing.client.error.log;

    root /usr/abills/cgi-bin;
    index index.cgi;

    location / {
        try_files $uri $uri/ /index.cgi$args;
    }

    location ~* /admin/ {
        allow 127.0.0.1;
        return 404;
        #deny all;
    }

    include perl-cgi.conf;
}
```
### Настройка хоста для админов
```bash
nano /etc/nginx/sites-available/admins.abills
```
```nginx
    listen 80;
    server_name admin.billing.net;

    #access_log /var/log/nginx/billing.admin.access.log;
    access_log off;
    error_log /var/log/nginx/billing.admin.error.log;

    root /usr/abills/cgi-bin/admin;
    index index.cgi;

    location / {
        try_files $uri $uri/ /index.cgi$args;
        deny 10.10.0.0/24; # Запрет доступа к админке абонентов с негативным депозитом
        deny 10.24.0.0/24; # Запрет доступа к админке абонентов с незарегистрированным оборудованием
    }

    location /img {
        alias /usr/abills/cgi-bin/img;
    }

    location /calendar.js {
        alias /usr/abills/cgi-bin/calendar.js;
    }

    location /functions.js {
        alias /usr/abills/cgi-bin/functions.js;
    }

    location /print.css {
        alias /usr/abills/cgi-bin/print.css;
    }

    location /js {
        alias /usr/abills/cgi-bin/js;
    }

    location /styles {
        alias /usr/abills/cgi-bin/styles;
    }
    
    location /favicon.ico {
        alias /usr/abills/cgi-bin/favicon.ico;
    }

    location /charts {
        alias /usr/abills/cgi-bin/charts;
    }

    location /graphics.cgi {
        alias /usr/abills/cgi-bin/graphics.cgi;
    }

    include perl-cgi.conf;
}
```
Активируем оба хоста:
```bash
ln -s /etc/nginx/sites-available/clients.abills /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/admins.abills /etc/nginx/sites-enabled/
```
Проверяем правильность конфигурации **nginx**:
```bash
nginx -t
```
Если все хорошо, то ответ будет следующим:
```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
И перечитываем конфиги **nginx**:
```bash
nginx -s reload
```
## IPTables
Если **iptables** настроен на запрет всех подключений, то нужно добавить пару правил:
```bash
iptables -t raw -A PREROUTING -i IF_NAME -s IP -p udp --dport 1812 -j ACCEPT
iptables -t raw -A PREROUTING -i IF_NAME -s IP -p udp --dport 1813 -j ACCEPT
```