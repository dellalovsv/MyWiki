---
title: Nginx + Php + MariaDB + PhpMyAdmin + Perl
description: Установка и настройка NGiNX + PHP + MariaDB + PhpMyAdmin + Perl на Debian
published: true
date: 2022-02-02T16:10:37.613Z
tags: debian, linux, mariadb, mysql, nginx, perl, php, phpmyadmin
editor: markdown
dateCreated: 2022-02-02T14:49:07.617Z
---

# Установка и настройка NGiNX + PHP + MariaDB + PhpMyAdmin + Perl
## Установка NGiNX, PHP
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
## Установка MariaDB, PhpMyAdmin
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
## Подключение Perl к NGiNX
### Установка
```bash
apt install libfcgi-perl -y
```
### Настройка
Нужно будет создать и заполнить несколько файлов.
Создаем **fastcgi-wrapper.pl** и заполняем его:
```bash
nano /usr/bin/fastcgi-wrapper.pl
```
```perl
#!/usr/bin/perl

use FCGI;
use Socket;
use POSIX qw(setsid);

require 'syscall.ph';

&daemonize;

#this keeps the program alive or something after exec'ing perl scripts
END() { } BEGIN() { }
*CORE::GLOBAL::exit = sub { die "fakeexit\nrc=".shift()."\n"; }; 
eval q{exit}; 
if ($@) { 
	exit unless $@ =~ /^fakeexit/; 
};

&main;

sub daemonize() {
    chdir '/'                 or die "Can't chdir to /: $!";
    defined(my $pid = fork)   or die "Can't fork: $!";
    exit if $pid;
    setsid                    or die "Can't start a new session: $!";
    umask 0;
}

sub main {
        $socket = FCGI::OpenSocket( "127.0.0.1:8999", 10 ); #use IP sockets
        $request = FCGI::Request( \*STDIN, \*STDOUT, \*STDERR, \%req_params, $socket );
        if ($request) { request_loop()};
            FCGI::CloseSocket( $socket );
}

sub request_loop {
        while( $request->Accept() >= 0 ) {
            
           #processing any STDIN input from WebServer (for CGI-POST actions)
           $stdin_passthrough ='';
           $req_len = 0 + $req_params{'CONTENT_LENGTH'};
           if (($req_params{'REQUEST_METHOD'} eq 'POST') && ($req_len != 0) ){ 
                my $bytes_read = 0;
                while ($bytes_read < $req_len) {
                        my $data = '';
                        my $bytes = read(STDIN, $data, ($req_len - $bytes_read));
                        last if ($bytes == 0 || !defined($bytes));
                        $stdin_passthrough .= $data;
                        $bytes_read += $bytes;
                }
            }

            #running the cgi app
            if ( (-x $req_params{SCRIPT_FILENAME}) &&  #can I execute this?
                 (-s $req_params{SCRIPT_FILENAME}) &&  #Is this file empty?
                 (-r $req_params{SCRIPT_FILENAME})     #can I read this file?
            ){
		pipe(CHILD_RD, PARENT_WR);
		my $pid = open(KID_TO_READ, "-|");
		unless(defined($pid)) {
			print("Content-type: text/plain\r\n\r\n");
                        print "Error: CGI app returned no output - ";
                        print "Executing $req_params{SCRIPT_FILENAME} failed !\n";
			next;
		}
		if ($pid > 0) {
			close(CHILD_RD);
			print PARENT_WR $stdin_passthrough;
			close(PARENT_WR);

			while(my $s = <KID_TO_READ>) { print $s; }
			close KID_TO_READ;
			waitpid($pid, 0);
		} else {
	                foreach $key ( keys %req_params){
        	           $ENV{$key} = $req_params{$key};
                	}
        	        # cd to the script's local directory
	                if ($req_params{SCRIPT_FILENAME} =~ /^(.*)\/[^\/]+$/) {
                        	chdir $1;
                	}

			close(PARENT_WR);
			close(STDIN);
			#fcntl(CHILD_RD, F_DUPFD, 0);
			syscall(&SYS_dup2, fileno(CHILD_RD), 0);
			#open(STDIN, "<&CHILD_RD");
			exec($req_params{SCRIPT_FILENAME});
			die("exec failed");
		}
            } 
            else {
                print("Content-type: text/plain\r\n\r\n");
                print "Error: No such CGI app - $req_params{SCRIPT_FILENAME} may not ";
                print "exist or is not executable by this process.\n";
            }

        }
}
```
Сохраняем, закрываем (<kbd>F2</kbd>, <kbd>CTRL+X</kbd>).

---

Создаем еще один файл **perl-fcgi** и заполняем его:
```bash
nano /etc/init.d/perl-fcgi
```
```bash
#!/bin/bash
### BEGIN INIT INFO
# Provides:          perl-fcgi
# Required-Start:    networking
# Required-Stop:     networking
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start the Perl FastCGI daemon.
### END INIT INFO 
PERL_SCRIPT=/usr/bin/fastcgi-wrapper.pl
FASTCGI_USER=www-data
RETVAL=0
case "$1" in
    start)
      su - $FASTCGI_USER -c $PERL_SCRIPT
      RETVAL=$?
  ;;
    stop)
      killall -9 fastcgi-wrapper.pl
      RETVAL=$?
  ;;
    restart)
      killall -9 fastcgi-wrapper.pl
      su - $FASTCGI_USER -c $PERL_SCRIPT
      RETVAL=$?
  ;;
    *)
      echo "Usage: perl-fcgi {start|stop|restart}"
      exit 1
  ;;
esac      
exit $RETVAL
```
Сохраняем, закрываем (<kbd>F2</kbd>, <kbd>CTRL+X</kbd>).

---

Для работы данных скриптов нужно установить **sudo** и внести изминения в файл **perl-fcgi**:
```bash
apt install sudo -y
sed -i -e 's/su\ -/sudo\ -u/g' -e '/sudo/s/-c\ //g' /etc/init.d/perl-fcgi
```
Далее даем права на запуск данных скриптов:
```bash
chmod +x /usr/bin/fastcgi-wrapper.pl
chmod +x /etc/init.d/perl-fcgi
```
Добавляем **perl-fcgi** в автозапуск и запускаем его:
```bash
systemctl enable perl-fcgi
/etc/init.d/perl-fcgi start
```
Теперь нужно добавить **location** в хост файл или создать отдельный файл и подключать его дерективой **include**:
```nginx
location ~ \.cgi$ { #(В оригинале location ~ \.pl$ {)
  try_files $uri =404;
	gzip off;
	fastcgi_pass  127.0.0.1:8999;
	fastcgi_index index.cgi; #(В оригинале fastcgi_index index.pl;)
	fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
	include fastcgi_params;
}
```

---

Перезагружаем **NGiNX**:
```bash
/etc/init.d/nginx restart
```