
---
title:zabbixのDBデータを最小限で抽出する
---
# zabbixのDBデータを最小限で抽出する

## 目的

- コンフィグバックアップの代わりにDBデータを採取したい

## 参考にしたURI


- Zabbix Backup
  
  https://zabbixbackup.com/pages/examples.html
- GitHub

  https://github.com/remontti/zabbix-backup


## MySQL/MariaDBの場合

- 安直に: mysqldump -u root -p zabbix

  アップグレードの種で使う場合はこんな感じ

  ```
  # 移行元
  # trends* と history* を除外してダンプする
  time mysqldump -u root -p zabbix --single-transaction \
    --ignore-table=zabbix.history \
    --ignore-table=zabbix.history_uint  \
    --ignore-table=zabbix.trends \
    --ignore-table=zabbix.trends_uint \
    --ignore-table=zabbix.history_str \
    --ignore-table=zabbix.history_log  |gzip -c > zabbix_conf.sql
  ```

  ```
  # 移行先
  # 予めアップグレード元のSQL を投入
  time zcat /usr/share/doc/zabbix-server-mysql-3.0.32/create.sql.gz | mysql -u root zabbix
  # エクスポートしたSQLを投入
  time zcat /var/tmp/somebackup.sql.gz | mysql -u root zabbix
  ```

- 安全方向に倒す場合: ZABBIXサポートに契約して、バックアップスクリプトをわけてもらう

- 海外の有志のスクリプトを使う

  おそらく5.0くらいなら問題なし

  アップグレードの種で使うと「バージョンテーブルから類推されるテーブルと実際のテーブルが違う」みたいな
  エラーがばんばんあがるので、あんまりおすすめしない

  https://github.com/remontti/zabbix-backup