---
title: Настройка L2TP+IPSec сервера на Debian
description: Настройка L2TP+IPSec сервера на Debian
published: false
date: 2022-02-02T19:01:10.148Z
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
Скачать исходники можно [здесь](https://files.delovoyadmin.net/l2tp/openswan-v2.6.52.3.tar.gz).
