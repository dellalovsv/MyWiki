---
title: WireGuard
description: Установка и настройка WireGuard на Debian 11
published: true
date: 2022-02-06T09:52:18.829Z
tags: debian, vpn, wireguard
editor: markdown
dateCreated: 2022-02-06T09:52:18.829Z
---

# Установка и настройка WireGuard
## Установка
```bash
apt install wireguard -y
```
Для последующего добавлении профиля в телефон используя QR код , установим пакеты mawk grep iproute2 qrencode
```bash
apt install mawk grep iproute2 qrencode -y
```
## Настройка
Настроить WireGuard нам поможет скрипт easy-wg-quick скачиваем его, даем права на выполнение и запускаем:
```bash
wget https://raw.githubusercontent.com/burghardt/easy-wg-quick/master/easy-wg-quick
chmod +x easy-wg-quick
./easy-wg-quick
```
Все необходимые настройки скрипт сделает за нас и по завершению вы получите QR код:
![wireguard_qrcode.png](/images/wireguard_qrcode.png)
### Подключение
Для добавления подключения на мобильных устройствам Вам нужно установить программу WireGuard для Andorid или iOS и войдя в приложение сканировать QR код который вы получили.

Для подключения к серверу использую компьютер под управление операционной системы Windows Вам нужно установить приложение WineGuard for Windows и после его установки нужно скопировать код туннеля в программу. Для этого нужно выполнить команду
```bash
cat wgclient_10.conf
```
Ответ:
```bash
# 10: 10 > wgclient_10.conf
[Interface]
Address = 10.127.0.10/24
DNS = 1.1.1.1
PrivateKey = WaDsq1e031849jB8QMlr314l;od%62!B1BpDC3L+CvY=

[Peer]
PublicKey = 0mbasfv56!fghhjDbCsasajkyujZmLi2werwei48iEu4fx0=
PresharedKey = bjbOtzase6z05s23QcVoiYtaPo1+QWz5SXHgnA232DD49q1@34Bf0=
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = 5.5.5.5:14302
PersistentKeepalive = 25
```
В случаи если вы создали дополнительный туннель то указать имя туннеля, имя туннеля начинается c **wgclient**.
В приложении для Windows нажимаем **Add Tennel - Add empty tunnel...** и вставить текст который получите после выполнения команды которая указана выше и задайте имя подключения. Для подключения к серверу **WireGuard** нужно нажать кнопку **Activate**.

Для добавления новых профилей нужно выполнить команду:
```bash
./easy-wg-quick имя_профиля
```
После того как вы закончили добавление профилей обновим конфигурацию сервера, включим сервер и добавим его в автозагрузку:
```bash
cp wghub.conf /etc/wireguard/wghub.conf 
systemctl enable wg-quick@wghub
systemctl start wg-quick@wghub
```
Для просмотра текущих подключений и статуса сервера выполните команду:
```bash
wg show
```
Все готово!