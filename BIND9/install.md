---
title: Установка
description: Установка и настройка BIND9 на Debian
published: false
date: 2022-02-02T19:40:35.912Z
tags: bind9, debian, dns, linux
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
Наполняем файл **named.conf.options**:
```bash
nano /etc/bind/named.conf.options
```
```bash
# Список интерфейсов/ip адресов на которых слушать
acl "listenOn" {
  127.0.0.1;
  X.X.X.X;
};

# Список доверенных ip адресов
acl "trusted" {
    localhost;
    127.0.0.1;
    X.X.X.X;
};

# Список ip адресов для внутренней зоны
acl "matchClientsInt" {
    127.0.0.1;
    X.X.X.X;
};

options {
    directory "/var/cache/bind";
    listen-on-v6 { none; };
    listen-on { listenOn; };
    allow-query-cache { trusted; };
    allow-query { trusted; };
    allow-recursion { trusted; };
    allow-notify { trusted; };
    allow-transfer { trusted; };
    recursive-clients 3000;
    clients-per-query 20;
    max-clients-per-query 30;
    version "Hitler Kaput!";
};

logging {
    channel null { null; };

    channel default_log {
        file "/var/log/named/named.log" versions 5 size 50M;
        print-time yes;
        print-severity yes;
        print-category yes;
    };

    category default { default_log; };
    category general { default_log; };
    category notify { null; };
    category lame-servers { null; };
    category edns-disabled { null; };
    category queries { null; };
};

view "internal" {
    match-clients { matchClientsInt; };

    allow-recursion { trusted; };

    zone "." in {
        type hint;
        file "/etc/bind/db.root";
    };

    zone "lab.local" {
        type master;
        file "/etc/bind/zones/ns1.lab.local.int";
    };

    include "/etc/bind/zones.rfc1918";
};

view "public" {
    match-clients { any; };
    allow-recursion { trusted; };
    allow-query { trusted; };

    zone "lab.local" {
        type master;
        file "/etc/bind/zones/ns1.lab.local";
    };
};
```