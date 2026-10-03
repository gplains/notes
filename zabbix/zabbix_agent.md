
---
title:Zabbix_agent
---
# Zabbix_agentの設定

だいたいGeminiの受け売り

## Windows/Linux共通

- 初期設定は zabbix_agent.conf
  
  Win: C:\Program Files\Zabbix Agent 2/zabbix_agent2.conf

  Linux: /etc/zabbix/zabbix_agent2.conf

- zabbix_agent2.conf を書き換える代わりに新規にファイルを置いて設定の上書きもできる

  初期ファイルはあえていじらないのが今風らしい
  
  Win: C:\Program Files\Zabbix Agent 2\zabbix_agent2.d/01-somefile.conf

  Linux: /etc/zabbix/zabbix_agent2.d/01-somefile.conf

- ワンライナーで一気に「上書き用ファイル」を作る方法

```
# 共通:Hostname区は各ホスト名を参照して書き換え、初期値はなぜかZabbix Serverなので被る

# Windows : PowerShell
"Server=192.168.1.100`nServerActive=192.168.1.100`nHostname=somehost" |`
  Out-File -Encoding ascii "C:\Program Files\Zabbix Agent 2\zabbix_agent2.d\override.conf";`
  Restart-Service "Zabbix Agent 2"

# Linux
echo -e "Server=192.168.1.100\nServerActive=192.168.1.100\nHostname=somehost" |\
 sudo tee /etc/zabbix/zabbix_agent2.d/override.conf > /dev/null \
 && sudo systemctl restart zabbix-agent2

```

## Windows限定

WindowsのMSI実行時に設定ファイルを入れ込める

```
msiexec /i "zabbix_agent2-6.0.0-windows-amd64-openssl.msi" /qn ^
  SERVER="192.168.1.100" ^
  SERVERACTIVE="192.168.1.100" ^
  HOSTNAME="%COMPUTERNAME%" ^
  ENABLEPATH="1" ^
  INSTALLFOLDER="C:\Program Files\Zabbix Agent 2"
```
