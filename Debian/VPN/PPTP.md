---
title: PPTP
description: Настройка PPTP сервера на Debian
published: true
date: 2022-02-02T18:57:02.711Z
tags: debian, pptp, pptpd, vpn
editor: markdown
dateCreated: 2022-02-02T16:16:19.886Z
---

# Настройка PPTP сервера на Debian
## Установка
```bash
apt install pptpd -y
```
## Настройка
### Добавление интерфейса
В файле **interfaces** дописываем следующее (Добавим еще один IP адрес на интерфейс)
```bash
nano /etc/network/interfaces
```
```bash
auto {IF_NAME}:1
iface {IF_NAME}:1 inet static
  address 10.100.0.1/24
```
Сохраняем, закрываем (<kbd>F2</kbd>, <kbd>CTRL+X</kbd>).
Теперь можем поднять данный интерфей:
```bash
ifup {IF_NAME}:1
```
### PPTPD
Если файл **pptpd.conf** существует, то очищаем его предварительно сделав копию:
```bash
cp /etc/pptpd.conf{,.bak}
echo '' > /etc/pptpd.conf
```
Открываем на редактирование и заполняем его:
```bash
nano /etc/pptpd.conf
```
```bash
option /etc/ppp/pptpd-options
logwtmp
# Интерфейс на котором будет шлюз
bcrelay eth0:1
# Адрес шлюза
localip 10.100.0.1
# Пул адресов клиентов
remoteip 10.100.0.2-254
```
Сохраняем, закрываем (<kbd>F2</kbd>, <kbd>CTRL+X</kbd>).

Далее создаем копию файла **options** и очищаем оригинал:
```bash
cp /etc/ppp/options{,.bak}
echo '' > /etc/ppp/options
```
Открываем файл и заполняем его:
```bash
nano /etc/ppp/options
```
```bash
# ms-dns: DNS выдаваемые клиентам
ms-dns 77.88.8.1
ms-dns 8.8.8.8
require-mschap-v2
asyncmap 0
auth
crtscts
lock
hide-password
modem
debug
name pptpd
proxyarp
lcp-echo-interval 10
lcp-echo-failure 100
noipx
# Не использовать данный сервер как маршрут по умолчанию.
nodefaultroute
```
Сохраняем, закрываем (<kbd>F2</kbd>, <kbd>CTRL+X</kbd>).

Создаем файл **pptpd-options** и заполняем его:
```bash
nano /etc/ppp/pptpd-options
```
```bash
name pptpd
refuse-pap
refuse-chap
refuse-mschap
require-mschap-v2
require-mppe-128
```
Добавляем сервис в автозапуск
```bash
systemctl enable pptpd
```
## Добавление клиентов
Открываем файл **chap-secrets** и дописываем:
```bash
nano /etc/ppp/chap-secrets
```
```bash
# client        server        secret                IP addresses
login           pptpd         password              10.100.0.2
# Что-бы клиент получал динамический IP адрес нужно просто указать *
```

---

Перезапускаем **pptpd**:
```bash
systemctl restart pptpd
```