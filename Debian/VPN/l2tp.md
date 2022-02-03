---
title: L2TP+IPSec
description: Настройка L2TP+IPSec сервера на Debian
published: false
date: 2022-02-03T08:29:14.881Z
tags: debian, ipsec, l2tp, vpn
editor: markdown
dateCreated: 2022-02-02T18:56:51.160Z
---

# Настройка L2TP+IPSec сервера
## Подготовка системы
```bash
apt install libgmp3-dev gawk flex bison make -y
```

## Установка xL2TPD
```bash
apt install xl2tpd -y
```

## Сборка OpenSwan
> Openswan ставим из исходников, поскольку нет кандидатов на установки из репозиториев и все заработало только с этой версией.
{.is-info}

Скачать исходники можно [здесь](https://github.com/xelerance/Openswan/archive/master.tar.gz).

Начинаем собирать:
```bash
tar -xvzf master.tar.gz
cd Openswan*
make programs
make install
```

## Настройка
## IPSec
Открываем файл **ipsec.conf**
```bash
nano /etc/ipsec.conf
```
> В файле не используются табуляции. Вместо них пробелы
{.is-warning}

Заполняем файл:
```bash
config setup
    nat_traversal=yes
    virtual_private=%v4:10.0.0.0/8,%v4:192.168.0.0/16,%v4:172.16.0.0/12
    oe=off
    protostack=netkey

conn L2TP-PSK-NAT
    rightsubnet=vhost:%priv
    also=L2TP-PSK-noNAT
conn L2TP-PSK-noNAT
    authby=secret
    pfs=no
    auto=add
    keyingtries=3
    rekey=no
    ikelifetime=8h
    keylife=1h
    type=transport
    # IP адрес внешнего интерфейса
    left=SERVER.IP
    leftprotoport=17/1701
    right=%any
    rightprotoport=17/%any
```
### Настройка авторизации
Открываем файл **ipsec.secrets**:
```bash
nano /etc/ipsec.secrets
```
Заполняем файл:
```bash
SERVER.IP %any: PSK "PreSharedKey"
```
> **SERVER.IP** — это внешний IP нашего сервера;
**%any:** — подключение с любого IP;
**PSK** — метод авторизации для L2TP;
**PSK — PreSharedKey** — вместо сертификата будем использовать секретное слово;
**«PreSharedKey»** — само секретное слово
{.is-info}

### Скрипт
```bash
nano /usr/local/sbin/ipsec
```
Содержимое:
```bash
#!/bin/bash

for each in /proc/sys/net/ipv4/conf/*
do
echo 0 > $each/accept_redirects
echo 0 > $each/send_redirects
done
/etc/init.d/ipsec restart
```
Даем права на запуск:
```bash
chmod +x /usr/local/sbin/ipsec
```
> Данный скрипт нужно добавить в автозагрузку
{.is-info}

## L2TP
Открываем файл **xl2tpd.conf**:
```bash
nano /etc/xl2tpd/xl2tpd.conf
```
Наполняем следующим:
```bash
[global]
ipsec saref = yes

[lns default]
ip range = 192.168.1.202-192.168.1.220
local ip = 192.168.1.201
refuse chap = yes
refuse pap = yes
require authentication = yes
ppp debug = yes
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes
```
Открываем файл **options.xl2tpd**:
```bash
nano /etc/ppp/options.xl2tpd
```
Наполняем следующим:
```bash
require-mschap-v2
ms-dns 192.168.1.1
asyncmap 0
auth
crtscts
lock
hide-password
modem
debug
name l2tpd
proxyarp
lcp-echo-interval 30
lcp-echo-failure 4
```
## Добавление клиентов
Открываем файл **chap-secrets**:
```bash
# client        server        secret                IP addresses
user		        l2tpd   	    password			        ip/* - for dynamic
# Что-бы клиент получал динамический IP адрес нужно просто указать *
```
## Перезапуск сервисов
```bash
/etc/init.d/ipsec restart
/etc/init.d/xl2tpd restart
```