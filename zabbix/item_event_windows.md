
---
title:ITEM_Event_Windows
---
# Windows エージェントでのイベントアイテムの考慮事項

- イベントアイテムの基本的な流れ

  アイテムの作成 (データ収集 > アイテム)

  トリガーの作成 (データ収集 > アイテム > 「トリガーの作成」)

- Windowsで「イベントIDベースで除外」を検討する場合

  正規表現(管理 > 一般設定 > 正規表現) から除外ルールを定義

## 正規表現の設定

- アイテムはApplication/Systemで最低2個欲しい

  例1: eventid_except_app

  例2: eventid_except_system

- 「すべての例外ルールに合致しなかったら成」の考え方を取る

  例: 

  - 条件式の形式: 結果が偽
  - 条件式: ^(10016|1289)$

## アイテムの設定

- タイプ:Zabbixエージェント(アクティブ)
- キー:

  ```eventlog[System,,"Warning|Error|Critical|Verbose",,"@eventid_except_system"]```

  まず、イベント件数が多いので Info/Debugはセベリティ(深刻度)から除外する

  正規表現リスト eventid_except_system で、「無視できる」イベントIDを除外する

- データ型:ログ

- 保存前処理

  イベントID観点の加工は難しそう(例えば1015は除外したい)

  あくまでテキスト観点(WSUSがログに入っていたら「正規表現と一致しない:WSUS」>「値を破棄」みたいな)でのログ加工は可能

## トリガーの設定

- 名前:任意
- イベント名:

  ```{ITEM.LOG.EVENTID} {ITEM.VALUE}```

  トリガー名ではなくてイベントID/イベント内容を出す場合はこんな感じで

- 条件式

  ```
  logeventid(/template/eventlog[System,,"Warning|Error|Critical|Verbose",,"@eventid_except_system"],,"@eventid_except_system")=1
  and logseverity(/template/eventlog[System,,"Warning|Error|Critical|Verbose",,"@eventid_except_system"])>=2
  ```

  logeventid について「すべての例外に合致しないこと」

  logseverity について 2以上であること 2:警告 4:エラー 9:重大

  