---
title: Установка/Перенос 0.59
description: Установка и настройка Abills 0.59 под NGiNX на Debian
published: false
date: 2022-02-02T16:46:13.155Z
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
## Настройка Freeradius

## Настройка NGiNX