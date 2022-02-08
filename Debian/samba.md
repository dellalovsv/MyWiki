---
title: Samba
description: Установка и настройка Samba на Debian
published: false
date: 2022-02-08T22:47:51.579Z
tags: debian, samba
editor: markdown
dateCreated: 2022-02-08T21:24:36.713Z
---

# Установка и настройка Samba
## Установка
```bash
apt install samba -y
```
> После установки в разных системах сервис **samba** может назваться по разному, например в debian **samba**, а на raspberry pi **smbd**.
{.is-warning}
## Настройка
> Вся конфигурация находится в файле **/etc/samba/smb.conf**.
{.is-info}
### Раздел [global]
```bash
[global]
# Рабочая группа в которой компьютеры смогут подключаться к samba
workgroup = WORKGROUP

# Возможнось ограничения доступа по маске сети или ip адресу
interfaces = 192.168.1.0/24 eth0

# Либо можно указать эту опцию и перечислить маски, адреса
hosts allow = 192.168.1.2 192.168.10.0/24
```
### Создание общего ресурса
