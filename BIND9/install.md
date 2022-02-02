---
title: Установка
description: Установка и настройка BIND9 на Debian
published: false
date: 2022-02-02T19:36:37.711Z
tags: debian, linux, bind9, dns
editor: markdown
dateCreated: 2022-02-02T19:36:37.711Z
---

# Установка и настройка Bind9
## Установка
```bash
apt install bind9 dnsutils -y
```
## Настройка
### Создание каталогов, файлов и выдача прав
Сразу создадим нужные каталоги и файлы:
```bash
mkdir /etc/bind/zones /var/log/named
touch /var/log/named/named.log
chown bind. /etc/bind/zones
chown -R bind. /var/log/named
```
### Конфигурируем
Для начала нужно создать бэкап оригинального конфига и очистим оригинал:
```bash
cp /etc/bind/named.conf.options{,.bak}
echo '' > /etc/bind/named.conf.options
```