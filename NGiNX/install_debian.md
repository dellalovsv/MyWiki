---
title: Nginx + Php + MariaDB + PhpMyAdmin + Perl
description: Установка и настройка NGiNX + PHP + MariaDB + PhpMyAdmin + Perl на Debian
published: false
date: 2022-02-02T14:50:57.361Z
tags: debian, linux, mariadb, mysql, nginx, perl, php, phpmyadmin
editor: markdown
dateCreated: 2022-02-02T14:49:07.617Z
---

# Установка и настройка NGiNX + PHP + MariaDB + PhpMyAdmin + Perl
## Установка
```bash
apt install nginx php-{fpm,common,mysql,gd,cli,curl,mbstring,cgi} mariadb-server mariadb-client phpmyadmin -y
```