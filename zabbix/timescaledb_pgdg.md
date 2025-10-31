
---
title:timescaledb_pgdg
---
# PGDGなPostgreSQLに対してTimescaleDBを適用する

ZabbixでTimescaleDBを使用するのはちょっとバージョン縛りがシビア

- 7.0.18-19

  PGDG版 PostgreSQL17 + PGDG版 TimescaleDB 2.21
- 7.0.20

  PGDG版 PostgreSQL18 + PGDG版 TimescaleDB 2.22

OSS版(PGDGでいうとPostgreSQLと同じパスにある)とTSL版(non-freeにある)の差異

- OSS版

  圧縮は使えない

  サービスとして提供する場合はこちらが利用可能

- TSL版

  圧縮が使える

  サービスとして提供する場合は利用不可

## 事前準備-7.0.19 かつ RHEL8 の場合

- PostgreSQLのリポジトリから以下のバイナリを入手する

  URI:  https://download.postgresql.org/pub/repos/yum/17/redhat/rhel-8-x86_64/

  postgresql17-libs-17.6-1PGDG.rhel8.x86_64.rpm

  postgresql17-server-17.6-1PGDG.rhel8.x86_64.rpm

  postgresql17-17.6-1PGDG.rhel8.x86_64.rpm

  URI:  https://download.postgresql.org/pub/repos/yum/non-free/17/redhat/rhel-8-x86_64/

  timescaledb-tsl_17-2.21.3-1PGDG.rhel8.x86_64.rpm

- RHEL8で dnf groupinstall するなりして以下のパッケージを適用する

  libicu-60.3-2.el8_1.x86_64.rpm

  libpq-13.5-1.el8.x86_64.rpm

## 実際-7.0.19 かつ RHEL8 の場合

以下、TSL版(timescaledb-tsl_17-2.21.3-1PGDG.rhel8.x86_64.rpm)を使用した場合の手順
OSS版(timescaledb_17-2.21.2-1PGDG.rhel8.x86_64.rpm)を使用した場合はrpmの箇所を読み替える

- rpmの適用

  ```
  sudo rpm -ivh timescaledb-tsl_17-2.21.3-1PGDG.rhel8.x86_64.rpm
  # 一旦スーパーユーザに切り替えて二行だけpostgresql.conf に追記
  sudo su - 
  echo "shared_preload_libraries = 'timescaledb'" >> /var/lib/pgsql/17/data/postgresql.conf
  echo "timescaledb.license_key='CommunityLicense'" >> /var/lib/pgsql/17/data/postgresql.conf
  exit
  ```

- あとは流れで
  
  ```
  sudo systemctl restart postgresql-17
  echo "CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;" | sudo -u postgres psql zabbix
  sudo cat /usr/share/zabbix-sql-scripts/postgresql/timescaledb/schema.sql | sudo -u zabbix psql 
  sudo du -m /var/lib/pgsql/
  sudo zabbix_server -R housekeeper_execute
  grep house /var/log/zabbix/zabbix_server.log
  ```

## OSS版(apacheライセンス)でtimescaledbを適用した場合

- メリット
  
  housekeepingが短縮される、はず(保持期間が1-2時間のデータはともかく、保持期間が31日とかのデータはチャンク削除で処理されるため)

- 考慮事項

  圧縮はされない

## TSLライセンスでtimescaledbを適用した場合

- メリット
  
  housekeepingが短縮される、はず(保持期間が1-2時間のデータはともかく、保持期間が31日とかのデータはチャンク削除で処理されるため)

  圧縮される(7日後に)

- 考慮事項

  DBaaS等ホスティング用途では使えない、基本そこは気にしなくてよい

