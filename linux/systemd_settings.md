
---
title:SYSTEMD_SETTINGS
---
# SYSTEMD周りの微調整

だいたい https://www.amazon.co.jp/dp/B0CP741RX8 にある内容のコピペ

## 設定ファイルの場所

- /usr/lib/systemd/system/someservice.service

  一見いじりたくなるがここのファイルは触ってはいけない

- /run/systemd/system/someservice.service

  システム起動時等でメモリ上？に展開される領域、保存されない

- /etc/systemd/system/somerservice.service

  /usr/lib/systemd/system からここにコピーしてみたりする、ややこしくなるかも

- /etc/systemd/system/someservice.service.d/override.conf

  /usr/lib/systemd/system/someservice.service から変更したい箇所をここに書いておくと幸せになれる

## override.conf の内容例

- zabbix_agent2の実行ユーザを変えたい

  ```
  sudo mkdir /etc/systemd/system/zabbix-agent2.service.d
  sudo vi /etc/systemd/system/zabbix-agent2.service.d/override.conf
  # 以下3行
  [Service]
  User=someuser
  Group=somegroup

  sudo systemctl daemon-reload
  sudo systemctl restart zabbix-agent2 ; journalctl -xe
  ```
