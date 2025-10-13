
---
title:ZABBIX API での template/host のエクスポート
---
# ZABBIX API での template/host のエクスポート

## 目的

- host/templateのRESTでのエクスポート

## 順序

- REST問い合わせするタネを作る

## REST問い合わせするタネを作る

- WebUI(フロントエンド) → ユーザー → APIトークン

## 実際

```
#!/bin/bash
 
# 出力先
exportto="出力先フォルダ"
 
# Zabbix APIログイン
header='Content-Type:application/json-rpc'
apiurl='https://zabbix2.g-plains.net/zabbix/api_jsonrpc.php'
zbxauth="<Frontendで生成したToken>"
 
# テンプレートグループを取得、ここでは"ORETEMPLATE" "ORETEMPLATE2" の決め打ち
json='{"jsonrpc": "2.0","method": "templategroup.get","params": {"output": "extend","filter":{ "name":[ "ORETEMPLATE","ORETEMPLATE2" ] } } ,"auth": "'${zbxauth}'","id": 1}'
groupids=$(curl -sS -X POST -H "${header}" -d "${json}" ${apiurl} --insecure | jq -r '.result[].groupid' | sed -e "s/^[0-9].*$/\"&\"/" | tr '\n' ','| sed -e "s/,$//")
 
# テンプレートを取得
json='{"jsonrpc": "2.0","method": "template.get","params": {"output": "extend", "groupids":[ '
json+=${groupids}
json+=' ]  },"auth": "'${zbxauth}'","id": 1}'
templateids=$(curl -sS -X POST -H "${header}" -d "${json}" ${apiurl} --insecure | jq -r '.result[].templateid'| sed -e "s/^[0-9].*$/\"&\"/" | tr '\n' ','| sed -e "s/,$//")
 
json='{"jsonrpc": "2.0","method": "configuration.export","params": {"options": { "templates":[ '
json+=${templateids}
json+=' ] }  , "format": "json"},  "auth": "'${zbxauth}'","id": 1}'
curl -sS -X POST -H "${header}" -d "${json}" ${apiurl} --insecure | jq -r ".result"| jq -r  > ${exportto}/template.txt
 
# ホスト一覧を取得 とにかく全部抜くのでホストグループは指定しない
json='{"jsonrpc": "2.0","method": "host.get","params": {"output": "extend"  },"auth": "'${zbxauth}'","id": 1}'
hostids=$(curl -sS -X POST -H "${header}" -d "${json}" ${apiurl} --insecure | jq -r '.result[].hostid'| sed -e "s/^[0-9].*$/\"&\"/" | tr '\n' ','| sed -e "s/,$//")
 
json='{"jsonrpc": "2.0","method": "configuration.export","params": {"options": { "hosts":[ '
json+=${hostids}
json+=' ] }  , "format": "json"},  "auth": "'${zbxauth}'","id": 1}'
curl -sS -X POST -H "${header}" -d "${json}" ${apiurl} --insecure | jq -r ".result"| jq -r  > ${exportto}/host.txt
 
exit 0
```