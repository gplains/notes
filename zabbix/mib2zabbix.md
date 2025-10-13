
---
title:MIB2ZABBIX
---
# MIB2ZABBIX

内容は一旦ブログ記事の転記

## mib2zabbix

- Perlベースのスクリプト
- SNMPWALKした結果を zabbixテンプレートとして出力
- 結果はXML形式
- ちょっと古い(ここ数年更新されてない)

## ブログ記事の中身

```
# 適当な手段で、 /usr/share/snmp/yamaha-private-mib 配下に mibファイル群を配置
echo "MIBDIRS /usr/share/snmp/mibs:/usr/share/snmp/yamaha-private-mibs" \
>>/usr/share/snmp/snmp.conf
echo MIBS all >> /usr/share/snmp/snmp.conf
cat /usr/share/snmp/snmp.conf
 
sudo dnf install "perl(SNMP)" "perl(XML::Simple)"
git clone https://github.com/zabbix-tools/mib2zabbix.git
 
snmptranslate 1.3.6.1.4.1.1182.2 #yamahaRT が表示されること
./mib2zabbix.pl -o 1.3.6.1.4.1.1182 > yamaha-mib.xml ; ls -l 
```
