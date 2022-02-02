---
title: Установка
description: Установка и настройка BIND9 на Debian
published: true
date: 2022-02-02T19:47:18.297Z
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
cp /etc/bind/db.local /etc/bind/zones/ns1.lab.local
chown -R bind. /etc/bind/zones
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
## Пример файла зоны
Пример файла **ns1.lab.local**:
```bash
nano /etc/bind/zones/ns1.lab.local
```
```bash
$TTL   86400 #Время обновления зоны в секундах (Сутки)
@       IN      SOA     ns1.lab.local. root.lab.local. (
                              2         ; Serial
                          43200         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      ns1.lab.local.

; Внешние IP-адреса
@               IN      A       X.X.X.X
www             IN      A       X.X.X.X
ns1             IN      A       X.X.X.X
```
### Применение настроек к Bind9
```bash
/etc/init.d/bind9 reload
```
### Настройка resolv.conf для использования локального DNS
```bash
nano /etc/resolv.conf
```
```bash
nameserver 127.0.0.1
domain lab.local
search lab.local
```