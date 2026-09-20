
---
title:SNMPTrapを採取する
---
# SNMPTrapを採取する

## 目的

- zabbix_server.conf に書いてあるのになるべく近い表現でSNMPTrapを採取する

## 参考にしたURI

### まずSNMPTrapのインストール

```
# snmptrapdのインストール
sudo dnf install net-snmp net-snmp-utils

# snmptrapd.conf にエントリ追加
sudo vi /etc/snmp/snmptrapd.conf
> traphandle default /bin/bash /usr/sbin/zabbix_trap_handler.sh 

# zabbix_trap_handler.sh
# 基本はこの辺のURIから取ってくる(バージョンによってかわりそう)
# https://github.com/zabbix/zabbix-docker/blob/7.4/templates/scripts/snmptraps/zabbix_trap_handler.sh
# /zabbix-docker > templates > scripts/snmptraps > zabbix_trap_handler.sh
sudo vi /usr/sbin/zabbix_trap_handler.sh
# 3行目を適当に修正
# ZABBIX_TRAPS_FILE="/var/log/snmptrap/snmptrap"
sudo chmod +x /usr/sbin/zabbix_trap_handler.sh

# snmptrapd 用のフォルダを切る
sudo setfacl -Rm g:zabbix:rx /var/log/
sudo setfacl -Rdm g:zabbix:rx /var/log/
sudo mkdir -p /var/log/snmptrap ; getfacl /var/log/snmptrap #snmptrap 配下も zabbix:r-x であること

# サービス有効化
sudo systemctl start snmptrapd
sudo systemctl enable snmptrapd

# zabbix-serverも一部修正が必要
sudo vi /etc/zabbix/zabbix_server.conf
# SNMPTrapperFile=/var/log/snmptrap/snmptrap に書き換え
sudo systemctl restart zabbix-server

# で、試験
snmptrap -v 2c -c public localhost "" netSnmpExampleNotification
tail /var/log/snmptrap/snmptrap
```

### ログローテーションも考えないと

snmptrap自体滅多に飛んでこないけれどログローテーションも一応実装する
```
sudo vi cat /etc/logrotate.d/snmptrap
# /var/log/snmptrap/snmptrap
# {
#     daily
#     rotate 14
#     dateext
#     missingok
#     notifempty
#     compress
#     create 0644 root root
#     sharedscripts
#     postrotate
#      /usr/bin/systemctl reload snmptrapd >/dev/null 2>&1 || true
#     endscript
# }
sudo logrotate -dv /etc/logrotate.d/snmptrap # テスト
sudo logrotate -v /etc/logrotate.d/snmptrap # 力づくで回したい人向け
```
