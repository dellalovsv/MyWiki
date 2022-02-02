---
title: Установка/Перенос 0.59
description: Установка и настройка Abills 0.59 под NGiNX на Debian
published: false
date: 2022-02-02T17:15:31.346Z
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
nano /etc/cronta
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

## Настройка Freeradius

## Настройка NGiNX

## IPTables
Если **iptables** настроен на запрет всех подключений, то нужно добавить пару правил:
```bash
iptables -t raw -A PREROUTING -i IF_NAME -s IP -p udp --dport 1812 -j ACCEPT
iptables -t raw -A PREROUTING -i IF_NAME -s IP -p udp --dport 1813 -j ACCEPT
```