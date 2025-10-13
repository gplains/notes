
---
title:Zabbixでの履歴テーブルの書き出し
---
# Zabbixでの履歴テーブルの書き出し(PostgreSQL)

## 目的

- 日次でhistory/trendsのエクスポートをするタネ

## 参考URI

- https://zabbixbackup.com/pages/examples.html
- https://gumfum.hatenablog.com/entry/2021/04/03/120000
- https://qiita.com/nakamasato/items/16945848d47659c7c2c9


## 雑なコマンド

- 一時ファイル出力先
  /tmp\psql_export
- crontab で仕込むコマンド
  cat /tmp/psql_export | sudo -u zabbix psql

一時ファイルの雑な定義
```
\copy (select *  from history where clock >=  1757948400 and clock < 1758034800 ) to '/tmp/20250916_history' with DELIMITER ',' csv header;
\copy (select *  from history_uint where clock >=  1757948400 and clock < 1758034800 ) to '/tmp/20250916_history_uint' with DELIMITER ',' csv header;
\copy (select *  from history_log where clock >=  1757948400 and clock < 1758034800 ) to '/tmp/20250916_history_log' with DELIMITER ',' csv header;
\copy (select *  from history_str where clock >=  1757948400 and clock < 1758034800 ) to '/tmp/20250916_history_str'  with DELIMITER ',' csv header;
\copy (select *  from history_text where clock >=  1757948400 and clock < 1758034800 ) to '/tmp/20250916_history_text'  with DELIMITER ',' csv header;
\copy (select *  from trends where clock >=  1757948400 and clock < 1758034800 ) to '/tmp/20250916_trends'  with DELIMITER ',' csv header;
\copy (select *  from trends_uint where clock >=  1757948400 and clock < 1758034800 ) to '/tmp/20250916_trends_uint'  with DELIMITER ',' csv header;
\q
```
ヘッダのないCSVファイルが生成される

epochtimeはGMTベースなので「前々日の15:00のepochtime」「前日の15:00のepochtime」をFROM/TOにそれぞれ当てはめる必要がある
