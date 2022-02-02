---
title: Nginx + Php + MariaDB + PhpMyAdmin + Perl
description: Установка и настройка NGiNX + PHP + MariaDB + PhpMyAdmin + Perl на Debian
published: false
date: 2022-02-02T15:13:49.926Z
tags: debian, linux, mariadb, mysql, nginx, perl, php, phpmyadmin
editor: markdown
dateCreated: 2022-02-02T14:49:07.617Z
---

# Установка и настройка NGiNX + PHP + MariaDB + PhpMyAdmin + Perl
## Установка
### NGiNX, PHP
```bash
apt install nginx php-{fpm,common,mysql,gd,cli,curl,mbstring,cgi} -y
```
#### Настройка NGiNX
В файле **nginx.conf** отключаем отображение версии и разрешаем длинные домены:
```bash
nano /etc/nginx/nginx.conf
```
Данные строки ниже нужно раскомментировать:
```bash
server_tokens off;
server_names_hash_bucket_size 64;
```
> Чтобы запретить доступ к сайту по IP адресу нужно настроить файл хоста **default**
{.is-info}

Делаем копию файла **default**:
```bash
cp /etc/nginx/sites-available/default{,.bak}
```
Очищаем файл **default**:
```bash
echo '' > /etc/nginx/sites-available/default
```
Открываем файл **default** на редактирование и заполняем:
```bash
nano /etc/nginx/sites-available/default
```
```bash
server {
  listen 80 default_server;
  server_name _;
  return 444;
}
```
#### Настройка PHP
Для запрета заливки посторонних скриптов и их выполения на свервер, отредактируем файл **/etc/php/x.x/fpm/php.ini**:
```bash
nano /etc/php/x.x/fpm/php.ini
```
Найдем дерективу **;cgi.fix_pathinfo=1** и приведем к виду:
```php
cgi.fix_pathinfo=0
```
Сохраняем, закрываем (<kbd>F2</kbd>, <kbd>CTRL+X</kbd>).