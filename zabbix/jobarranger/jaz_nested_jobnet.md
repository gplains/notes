
---
title:JAZ_NESTED_JOBNET
---
# ジョブネット同士のネスト

Job Arranger for Zabbix で想定されるケース

- 同じサーバにほぼ同じコマンドを実行する

  ```
  # job1
  /opt/jobcmd/command_arg1.sh
  # job2
  /opt/jobcmd/command_arg2.sh
  ```

## テンプレート用ジョブネットの作成

- 非公開ジョブネット にジョブネットを作る

- STARTアイコン、JOBアイコン、ENDアイコンを普通に作る

- JOBアイコンでジョブ変数に以下のように設定する

  変数名:適当な変数名(例えば LOCALARG)

  値: 上位側で定義するジョブコントローラ変数(例えば $GLOBALARG)

  実行: /opt/jobcmd/command_${LOCALARG}.sh

- 呼び出し先ジョブネットは忘れずに「有効」にしておく

## 公開ジョブネットの作成

- 公開ジョブネットにジョブネットを作る

- STARTアイコン、VARIABLEアイコン、JOBNETアイコン、ENDアイコンを普通に作る

  JOBNETアイコン:ジョブコントローラ変数を引き継いでいい感じに動作する

  TASKアイコン:ジョブコントローラ変数を引き継がずに動作する

- VARIABLEアイコンでジョブコントローラ変数に以下のように設定する

  変数名: 持ち回りするジョブコントローラ変数(非公開ジョブネットに合わせて GLOBALARG)

  値: 実際の値 (job1 の場合は arg1 だし、 job2 の場合は arg2 )
