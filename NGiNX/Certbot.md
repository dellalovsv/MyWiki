---
title: Certbot
description: Установка SSL сертификатов через CERTBOT в NGiNX
published: true
date: 2022-02-02T11:30:59.151Z
tags: nginx, ssl, https, certbot, letsencrypt, debian, linux
editor: markdown
dateCreated: 2022-02-02T11:22:43.149Z
---

# Установка SSL сертификата в NGiNX
## Установка
```
apt install certbot python3-certbot-nginx
```

## Настройка
### Получение сертификата для доменов/поддоменов
> Чтобы получить сертификаты сразу для нескольких доменов/поддоменов и нужно указать через пробел с ключем **-d**
{.is-info}

```
certbot --nginx -d yourdomain.com
```
### Автоматизация обновления сертификатов
В файле **crontab** в конце дописываем строку:
```
1 */12 * * *    root    certbot renew --quiet
```
Применяем настройка файла **crontab**:
```
crontab /etc/crontab
```