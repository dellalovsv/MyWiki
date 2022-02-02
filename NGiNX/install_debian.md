---
title: Nginx + Php + MariaDB + PhpMyAdmin + Perl
description: Установка и настройка NGiNX + PHP + MariaDB + PhpMyAdmin + Perl на Debian
published: false
date: 2022-02-02T15:47:35.337Z
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
Для запрета заливки посторонних скриптов и их выполения на свервер, отредактируем файл **/etc/php/{VERSION}/fpm/php.ini**:
```bash
nano /etc/php/{VERSION}/fpm/php.ini
```
Найдем дерективу **;cgi.fix_pathinfo=1** и приведем к виду:
```php
cgi.fix_pathinfo=0
```
Сохраняем, закрываем (<kbd>F2</kbd>, <kbd>CTRL+X</kbd>).
##### Подключаем PHP к NGiNX
Чтобы не писать одно и то же в каждом хост файле где нужно использовать **php**, заведем отдельный файл в директории **nginx**:
```bash
touch /etc/nginx/php-cgi.conf
```
**Через сокет**
Это пишем в ранее созданый файл **php-cgi.conf**
```nginx
location ~ \.php$ {
  try_files $fastcgi_script_name =404;
  fastcgi_index index.php;
  fastcgi_param script_FILENAME /scripts$fastcgi_script_name;
  include fastcgi.conf;
  include fastcgi_params;
  fastcgi_pass unix:/var/run/php/php{VERSION}-fpm.sock;
}
```
**Через TCP**
Открываем файл **/etc/php/{VERSION}/fpm/pool.d/www.conf**, находим дерективу **listen** и приводим к виду:
```php
listen = 127.0.0.1:9000
```
Это пишем в ранее созданый файл **php-cgi.conf**
```nginx
location ~ \.php$ {
  try_files $fastcgi_script_name =404;
  fastcgi_index index.php;
  fastcgi_param script_FILENAME /scripts$fastcgi_script_name;
  include fastcgi.conf;
  include fastcgi_params;
  fastcgi_pass 127.0.0.1:9000;
}
```
Перезагружаем **php-fpm**:
```bash
/etc/init.d/php{VERSION}-fpm restart
```
### MariaDB, PhpMyAdmin
#### Установка
```bash
apt install mariadb-server mariadb-client phpmyadmin -y
```
> В процессе установке нужно будет настроить БД для phpmyadmin (указать пароль). При запросе для какого веб-сервера нужно настроить хост, то нужно отказаться.
{.is-info}

#### Настройка MariaDB
##### Debian 9
```bash
mysqladmin -u root password 'enter password here'
service mariadb restart
mysql -u root -p mysql (Ввод пароля не требуется)
update user set plugin="" where user='root';
flush privileges;
exit
```
##### Debian 11
Нужно запусть **mysql_secure_installation**:
```bash
mysql_secure_installation
```
В процессе выполнения скрипта нужно будет ответить на вопросы.
#### Настройка PhpMyAdmin
Подключаем **PhpMyAdmin** к **NGiNX** для доступа к БД через веб.
Создаем файл **pma** и заполняем его:
```bash
nano /etc/nginx/sites-available/pma
```
```nginx
server {
  listen 80;
  server_name pma.yourhost.com;
  
  access_log off;
  error_log /var/log/nginx/pma.error.log;
  
  root /usr/share/phpmyadmin;
  index index.php;
  
  location / {
  	try_files $uri $uri/ /index.php?$args;
  }
  
  include php-cgi.conf;
}
```
Сохраняем, закрываем (<kbd>F2</kbd>, <kbd>CTRL+X</kbd>).

Активируем хост:
```bash
ln -s /etc/nginx/sites-available/pma /etc/nginx/sites-enabled/
```
Проверяем правильность конфигураций **nginx**:
```bash
nginx -t
```
Если все хорошо, то ответ будет следующим:
```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
Теперь можем перечитать конфиги **nginx**:
```bash
nginx -s reload
```