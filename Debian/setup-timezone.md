---
title: Настройка часового пояса
description: Настройка часового пояса с помощью команды timedatectl
published: true
date: 2022-02-02T21:07:33.409Z
tags: debian, timezones, timedatectl, часовой, пояс
editor: markdown
dateCreated: 2022-02-02T21:07:33.409Z
---

# Настройка часового пояса с помощью команды timedatectl
Чтобы посмотреть текущий часовой пояс:
```bash
timedatectl
```
```bash
# Результат команды
               Local time: Ср 2022-02-02 23:02:14 EET
           Universal time: Ср 2022-02-02 21:02:14 UTC
                 RTC time: Ср 2022-02-02 21:02:14
                Time zone: Europe/Zaporozhye (EET, +0200)
System clock synchronized: no
              NTP service: n/a
          RTC in local TZ: no
```

Чтобы посмотресь все доступные часовые пояса:
```bash
timedatectl list-timezones
```
```bash
# Результат команды
Africa/Abidjan
Africa/Accra
Africa/Algiers
Africa/Bissau
Africa/Cairo
Africa/Casablanca
Africa/Ceuta
Africa/El_Aaiun
Africa/Johannesburg
...
```

Чтобы установить часовой пояс:
```bash
timedatectl set-timezone Europe/Kiev
```
**Все! Часовой пояс установлен.**