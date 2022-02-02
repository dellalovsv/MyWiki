---
title: Accel-PPP Установка Debian
description: Установка и настройка Accel-PPP + Abills для IPoE на Debian
published: true
date: 2022-02-02T18:04:14.796Z
tags: abills, accel-ipoe, accel-ppp, debian, linux, nas, router
editor: markdown
dateCreated: 2022-02-02T11:55:30.601Z
---

# Установка и настройка Accel-PPP + Abills 0.59
Исходный код скачать можно ниже по ссылкам:
[Исходники на GitHub](https://github.com/accel-ppp/accel-ppp.git)

Скрипты:
set_irq_affinity
smp.sh
vlan.sh

## Установка
### Подготовка системы
```bash
apt update
cd /usr/src
apt install make cmake libcrypto++-dev libssl-dev libpcre3 libpcre3-dev git lua5.1 liblua5.1-0-dev vlan -y
apt install linux-headers-`uname -r` -y
reboot
```
### Скачиваем и распаковываем архив
```bash
cd /usr/src
mkdir accel-ppp
git clone https://github.com/accel-ppp/accel-ppp.git ./accel-ppp
```
### Сборка
```bash
mkdir accel-ppp-build
cd accel-ppp-build
cmake -DCMAKE_INSTALL_PREFIX=/usr/local -DKDIR=/usr/src/linux-headers-`uname -r` -DRADIUS=TRUE -DSHAPER=TRUE -DLOG_PGSQL=FALSE -DLUA=TRUE -DBUILD_IPOE_DRIVER=TRUE ../accel-ppp
make
make install
```
После сборки и установки нужно подключить драйвер **ipoe.ko**:
```shell
insmod /usr/src/accel-ppp-build/drivers/ipoe/driver/ipoe.ko
```

## Настройка
Создаем скрипт автозапуска:
```bash
nano /etc/init.d/accel-ppp
```
Заполняем следующим содержимым:
```bash
#!/bin/sh
# /etc/init.d/accel-ppp: set up the accel-ppp server
### BEGIN INIT INFO
# Provides:          accel-ppp
# Required-Start:    $networking
# Required-Stop:     $networking
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
### END INIT INFO

set -e

PATH=/bin:/usr/bin:/sbin:/usr/sbin:/usr/local/sbin;
ACCEL_PPTPD=`which accel-pppd`
. /lib/lsb/init-functions

if test -f /etc/default/accel-ppp; then
    . /etc/default/accel-ppp
fi

if [ -z $ACCEL_PPPTD_OPTS ]; then
  ACCEL_PPTPD_OPTS="-c /etc/accel-ppp.conf"
fi

case "$1" in
  start)
        log_daemon_msg "Starting accel-ppp server" "accel-pppd"
        if [ x`lsmod |awk /ipoe/'{print $1}'` = x ]; then
          insmod /usr/src/accel-ppp-build/drivers/ipoe/driver/ipoe.ko
        fi
        if start-stop-daemon --start --quiet --oknodo --exec $ACCEL_PPTPD -- -d -p /var/run/accel-pppd.pid $ACCEL_PPTPD_OPTS; then
            log_end_msg 0
        else
            log_end_msg 1
        fi
  ;;
  restart)
        log_daemon_msg "Restarting accel-ppp server" "accel-pppd"
        if [ x`lsmod |awk /ipoe/'{print $1}'` = x ]; then
          insmod /usr/src/accel-ppp-build/drivers/ipoe/driver/ipoe.ko
        fi
        start-stop-daemon --stop --quiet --oknodo --retry 180 --pidfile /var/run/accel-pppd.pid
        if start-stop-daemon --start --quiet --oknodo --exec $ACCEL_PPTPD -- -d -p /var/run/accel-pppd.pid $ACCEL_PPTPD_OPTS; then
            log_end_msg 0
        else
            log_end_msg 1
        fi
  ;;

  stop)
        log_daemon_msg "Stopping accel-ppp server" "accel-pppd"
        start-stop-daemon --stop --quiet --oknodo --retry 180 --pidfile /var/run/accel-pppd.pid
        log_end_msg 0
  ;;

  status)
    do_status
  ;;
  *)
    log_success_msg "Usage: /etc/init.d/accel-ppp {start|stop|status|restart}"
    exit 1
    ;;
esac

exit 0
```
Делаем файл исполняемым/даем права на запуск:
```bash
chmod +x /etc/init.d/accel-ppp
```
### LUA
Создаем файл **lua**.
```bash
nano /etc/accel-ppp.lua
```
Заполняем следующим содержимым:
```lua
function username(pkt)
return pkt:hdr('chaddr')
end
```
### Ротация логов
Создаем файл **accel-ppp**
```bash
nano  /etc/logrotate.d/accel-ppp
```
Заполняем следующим содержимым:
```bash
/var/log/accel-ppp/*.log {
      rotate 7
      daily
      size=100M
      compress
      missingok
      sharedscripts
      postrotate
              test -r /var/run/accel-pppd.pid && kill -HUP `cat /var/run/accel-pppd.pid`
      endscript
}
```
### Freeradius
Открываем файл **dictionary** на редактирование
```bash
nano /usr/local/share/accel-ppp/radius/dictionary
```
Дописываем в конце файла:
```bash
ATTRIBUTE DHCP-Router-IP-Address 241 ipaddr
ATTRIBUTE DHCP-Mask              242 integer
ATTRIBUTE L4-Redirect      243 integer
ATTRIBUTE L4-Redirect-ipset      244 string
ATTRIBUTE DHCP-Option82          245 octets
# Limit session traffic
ATTRIBUTE Session-Octets-Limit 227 integer
# What to assume as limit - 0 in+out, 1 in, 2 out, 3 max(in,out)
ATTRIBUTE Octets-Direction 228 integer
# Connection Speed Limit
ATTRIBUTE PPPD-Upstream-Speed-Limit 230 integer
ATTRIBUTE PPPD-Downstream-Speed-Limit 231 integer
ATTRIBUTE PPPD-Upstream-Speed-Limit-1 232 integer
ATTRIBUTE PPPD-Downstream-Speed-Limit-1 233 integer
ATTRIBUTE PPPD-Upstream-Speed-Limit-2 234 integer
ATTRIBUTE PPPD-Downstream-Speed-Limit-2 235 integer
ATTRIBUTE PPPD-Upstream-Speed-Limit-3 236 integer
ATTRIBUTE PPPD-Downstream-Speed-Limit-3 237 integer
ATTRIBUTE Acct-Interim-Interval 85 integer
ATTRIBUTE Acct-Input-Gigawords    52      integer
ATTRIBUTE Acct-Output-Gigawords   53      integer
```
Так-же нужно добавить следующие в словарь **Freeradius**
```bash
# Freeradius v2.x
nano /etc/freeradius/dictionary

# Freeradius v3.x
nano /etc/freeradius/3.x/dictionary
```
Дописываем в конце файла:
```bash
ATTRIBUTE DHCP-Router-IP-Address 241 ipaddr
ATTRIBUTE DHCP-Mask              242 integer
ATTRIBUTE L4-Redirect      243 integer
ATTRIBUTE L4-Redirect-ipset      244 string
ATTRIBUTE DHCP-Option82          245 octets
# Limit session traffic
ATTRIBUTE Session-Octets-Limit 227 integer
# What to assume as limit - 0 in+out, 1 in, 2 out, 3 max(in,out)
ATTRIBUTE Octets-Direction 228 integer
# Connection Speed Limit
ATTRIBUTE PPPD-Upstream-Speed-Limit 230 integer
ATTRIBUTE PPPD-Downstream-Speed-Limit 231 integer
ATTRIBUTE PPPD-Upstream-Speed-Limit-1 232 integer
ATTRIBUTE PPPD-Downstream-Speed-Limit-1 233 integer
ATTRIBUTE PPPD-Upstream-Speed-Limit-2 234 integer
ATTRIBUTE PPPD-Downstream-Speed-Limit-2 235 integer
ATTRIBUTE PPPD-Upstream-Speed-Limit-3 236 integer
ATTRIBUTE PPPD-Downstream-Speed-Limit-3 237 integer
ATTRIBUTE Acct-Interim-Interval 85 integer
ATTRIBUTE Acct-Input-Gigawords    52      integer
ATTRIBUTE Acct-Output-Gigawords   53      integer
```
### Конфигурация Accel-PPP
Делаем копию оригинала (если он существует), очищаем оригинал:
```bash
cp /etc/accel-ppp.conf{,.bak}

# Если файл не существует, то создаем новый
nano /etc/accel-ppp.conf
```
Заполняем следующим:
```bash
[modules]
log_file
ipoe
radius
shaper
#
[core]
log-error=/var/log/accel-ppp/core.log
thread-count=4
#
[common]
#single-session=replace
#sid-case=upper
#sid-source=seq
#
[ppp]
verbose=1
min-mtu=1280
mtu=1400
mru=1400
#accomp=deny
#pcomp=deny
#ccp=0
#check-ip=0
#mppe=require
ipv4=require
ipv6=deny
ipv6-intf-id=0:0:0:1
ipv6-peer-intf-id=0:0:0:2
ipv6-accept-peer-intf-id=1
lcp-echo-interval=20
#lcp-echo-failure=3
lcp-echo-timeout=120
unit-cache=1
#
[auth]
#any-login=0
#noauth=0
#
[pptp]
verbose=1
#echo-interval=30
#
[pppoe]
verbose=1
#ac-name=xxx
#service-name=yyy
#pado-delay=0
#pado-delay=0,100:100,200:200,-1:500
called-sid=mac
#tr101=1
#padi-limit=0
#ip-pool=pppoe
#interface=eth1,padi-limit=1000
#sid-uppercase=0
interface=eth0
#
[l2tp]
verbose=1
#dictionary=/usr/local/share/accel-ppp/l2tp/dictionary
#hello-interval=60
#timeout=60
#rtimeout=1
#rtimeout-cap=16
#retransmit=5
#recv-window=16
#host-name=accel-ppp
#dir300_quirk=0
#secret=
#dataseq=allow
#reorder-timeout=0
#ip-pool=l2tp
#
[ipoe]
nas-identifier=nas-1
verbose=1
username=lua:username
password=empty
lua-file=/etc/accel-ppp.lua
lease-time=300
renew-time=150
max-lease-time=600
#offer-delay=200:100,300:200
attr-dhcp-client-ip=Frame-IP-Address
attr-dhcp-router-ip=DHCP-Router-IP-Address
attr-dhcp-mask=DHCP-Mask
attr-l4-redirect=L4-Redirect

### Здесь нужно указать шлюзы для всех подсетей которые будут использоваться
### Public
gw-ip-address=X.X.X.X/X



#
check-mac-change=0
proxy-arp=1
shared=1
ifcfg=1
mode=L2
start=dhcpv4
attr-dhcp-opt82=DHCP-Option82
#
#### Interfaces на которых Accel-ppp будет слушать запросы
interface=eth1.2
#
#### Вставляем свои DNS сервера
[dns]
dns1=1.1.1.1
dns2=8.8.8.8
#
[wins]
#wins1=172.16.0.1
#wins2=172.16.1.1
#
[radius]
dictionary=/usr/local/share/accel-ppp/radius/dictionary
# NAS
nas-identifier=X.X.X.X
# NAS
nas-ip-address=X.X.X.X
# NAS
gw-ip-address=X.X.X.X
# BILLING
auth-server=X.X.X.X:1812,blablabla
# BILLING
acct-server=X.X.X.X:1813,blablabla
# NAS
dae-server=X.X.X.X:3799,blablabla
verbose=1
interim-verbose=1
timeout=3
max-fail=10
max-try=3
acct-on=1
req-limit=10
fail-timeout=3
acct-timeout=0
##########################
#server=46.160.78.114,blablabla,auth-port=1812,acct-port=1813,req-limit=50,fail-timeout=0,max-fail=10,weight=1
#dae-server=46.160.78.87:3799,blablabla
#verbose=6
#timeout=3
#max-try=3
#acct-on=1
#acct-interim-interval=600
#acct-timeout=0
#attr-tunnel-type=NAS-Identifier
#
[client-ip-range]
#10.0.0.0/8
#
[ip-pool]
gw-ip-address=10.0.0.1
#vendor=Cisco
#attr=Cisco-AVPair
attr=Framed-Pool
10.0.0.1-3.10,name=Main
#192.168.0.2-255
#192.168.1.1-255,name=pool1
#192.168.2.1-255,name=pool2
#192.168.3.1-255,name=pool3
#192.168.4.0/24
#
[log]
log-file=/var/log/accel-ppp/accel-ppp.log
log-emerg=/var/log/accel-ppp/emerg.log
log-fail-file=/var/log/accel-ppp/auth-fail.log
#log-debug=/dev/stdout
#syslog=accel-pppd,daemon
#log-tcp=127.0.0.1:3000
copy=1
color=1
#per-user-dir=per_user
#per-session-dir=per_session
#per-session=1
level=5
#
[log-pgsql]
conninfo=user=log
log-table=log
#
[pppd-compat]
#ip-pre-up=/etc/ppp/ip-pre-up
ip-up=/etc/ppp/ip-up
ip-down=/etc/ppp/ip-down
ip-change=/etc/ppp/ip-change
radattr-prefix=/var/run/radattr
verbose=1
#
[chap-secrets]
gw-ip-address=192.168.100.1
#chap-secrets=/etc/ppp/chap-secrets
#encrypted=0
#username-hash=md5
#
[shaper]
attr=Filter-Id
ifb=ifb0
up-limiter=htb
down-limiter=htb
cburst=1375000
r2q=10
quantum=1500
leaf-qdisc=sfg perturb 10
verbose=0
attr-down=PPPD-Downstream-Speed-Limit
attr-up=PPPD-Upstream-Speed-Limit
#
#attr=Filter-Id
#down-burst-factor=0.1
#up-burst-factor=1.0
#latency=50
#mpu=0
#mtu=0
#r2q=10
#quantum=1500
#moderate-quantum=1
#cburst=1534
#ifb=ifb0
#up-limiter=police
#down-limiter=tbf
#leaf-qdisc=sfq perturb 10
#leaf-qdisc=fq_codel [limit PACKETS] [flows NUMBER] [target TIME] [interval TIME] [quantum BYTES] [[no]ecn]
#rate-multiplier=1
#fwmark=1
#verbose=1
#
[cli]
verbose=1
telnet=127.0.0.1:2000
tcp=127.0.0.1:2001
#password=123
#
[snmp]
master=0
agent-name=accel-ppp
#
[connlimit]
limit=10/min
burst=3
timeout=60
#
[ipv6-pool]
fc00:0:1::/48,64
delegate=fc00:1::/36,48
#
[ipv6-dns]
#fc00:1::1
#fc00:1::2
#fc00:1::3
#dnssl=suffix1.local.net
#dnssl=suffix2.local.net.
#
[ipv6-dhcp]
verbose=1
pref-lifetime=604800
valid-lifetime=2592000
route-via-gw=1
```
### VLAN's
Скрипт для поднятия vlan'ов при запуске сервера
```bash
nano /path/to/script/vlan.sh
```
```bash
#!/bin/bash

IFACE="eth1"
VLANS="2-500"

/sbin/vconfig set_name_type DEV_PLUS_VID_NO_PAD
VLANS=`echo ${VLANS} | sed 'N;s/\n/ /' |sed 's/,/ /g'`
for i in $VLANS; do
  if [[ $i =~ - ]]; then
    IFS='-' read -a start_stop <<< "$i"
    for cur_iface in `seq ${start_stop[0]} ${start_stop[1]}`;
    do
      echo "${cur_iface}";
      /sbin/vconfig add ${IFACE} ${cur_iface}
      /bin/ip li set up ${IFACE}.${cur_iface}
    done
  else
  echo "$i";
    /sbin/vconfig add ${IFACE} ${i}
    /bin/ip li set up ${IFACE}.${i}
  fi;
done
```
Сохраняем (<kbd>f2</kbd>, <kbd>ctrl+x</kbd>) и даем права на запуск:
```bash
chmod +x /path/to/script/vlan.sh
```
### Автозагрузка через cronta + скрипт
Создаем файл скрипта автозагрузки:
```bash
nano /path/to/script/start_server.sh
```
```bash
#!/bin/sh

sysctl -w net.netfilter.nf_conntrack_max=1048576
echo 131072 > /sys/module/nf_conntrack/parameters/hashsize

/usr/local/sbin/vlan.sh
killall irqbalance
/usr/local/sbin/set_irq_affinity eth0
/usr/local/sbin/set_irq_affinity eth1
iptables-restore < /etc/iptables/rules.v4

/etc/init.d/accel-ppp start
```
В конце файла **crontab** дописываем:
```bash
@reboot  /bin/sleep 5 && /path/to/script/start_server.sh
```
Применяем настройки **crontab**:
```bash
crontab /etc/crontab
```
### IpTables
Добавляем заглушки:
```bash
iptables -A PREROUTING -s 10.0.0.0/22 ! -d 172.17.253.1/32 -p tcp -m tcp --dport 80 -j DNAT --to-destination 172.17.253.1:8080
iptables -A PREROUTING -s 10.24.0.0/22 ! -d 172.17.253.1/32 -p tcp -m tcp --dport 80 -j DNAT --to-destination 172.17.253.1:8002
```
Добавляем NAT:
```bash
# Клиентская подсеть
iptables -A POSTROUTING -s 172.16.50.0/24 -j SNAT --to-source EXT_IP
# Подсеть с негативным депозитом
iptables -A POSTROUTING -s 10.0.0.0/22 -d 172.17.253.1/32 -j SNAT --to-source EXT_IP
# Гостевая подсеть натит на заглушку
iptables -A POSTROUTING -s 10.24.0.0/22 -d INT_IP -j SNAT --to-source EXT_IP
```