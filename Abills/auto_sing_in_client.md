---
title: Вход абонента в ЛК без пароля
description: Автоматических вход Абонента в личный кабинет без запроса Логина и Пароля
published: true
date: 2022-02-02T18:13:54.564Z
tags: abills, автовход, вход без пароля
editor: markdown
dateCreated: 2022-02-02T18:10:56.613Z
---

# Вход Абонента в личный кабинет без запроса Логина и Пароля
В файле **config.pl** нужно дописать дерективу в любое место или привести ее к виду (если её там нет):
```bash
nano /path/to/abills/libexec/config.pl
```
```perl
$conf{PASSWORDLESS_ACCESS}=1;
```