
---
title:公式テンプレートからLLD設定をぶっこ抜く
---
# 公式テンプレートからLLD設定をぶっこ抜く

内容は一旦ブログ記事の転記


## 以下ブログ記事

- まず、ホストテンプレートに公式テンプレートからマクロを転記する
  ファイルシステム周りだと次の４つを使用するので転記
  ```
  {$VFS.FS.FSNAME.MATCHES}
    .+
  {$VFS.FS.FSNAME.NOT_MATCHES}
    ^(/dev|/sys|/run|/proc|.+/shm$)
  {$VFS.FS.FSTYPE.MATCHES}
    ^(btrfs|ext2|ext3|ext4|reiser|xfs|ffs|ufs|jfs|jfs2|vxfs|hfs|apfs|refs|ntfs|fat32|zfs)$
  {$VFS.FS.FSTYPE.NOT_MATCHES}
    ^\s$
  ```
- ホストテンプレートにアイテム「vfs.fs.get」を追加する
  保持期間は1日くらい
- 「テンプレートに絶対存在する」パーティションのアイテムとトリガーを追加する
  LLDで見つけてくれなさそうなエントリを粛々と追加する
- ディスカバリルール「vfs.fs.dependent.discovery」を追加する
  「依存アイテム」として、マスターアイテムに先ほどの「vfs.fs.get」のエントリを指定
   保存前処理、LLDマクロ、フィルタ、オーバーライド の各タブはそれぞれ公式テンプレートのそれを転記する
   ```
   # 保存前処理 > JavaScript
   var filesystems = JSON.parse(value);
   
   result = filesystems.map(function (filesystem) {
   	return {
  		'fsname': filesystem.fsname,
		'fstype': filesystem.fstype
	};
   });
   return JSON.stringify(result);
   # 保温前処理 > 指定秒内に変化がなければ破棄: 1h
   ```
   ```
   # LLDマクロ
   {#FSNAME} > $.fsname
   {#FSTYPE} > $.fstype
   ```
   ```
   # フィルタ
   {#FSNAME} 一致する   {$VFS.FS.FSNAME.MATCHES}
   {#FSNAME} 一致しない {$VFS.FS.FSNAME.NOT_MATCHES}
   {#FSTYPE} 一致する   {$VFS.FS.FSTYPE.MATCHES}
   {#FSTYPE} 一致しない  {$VFS.FS.FSTYPE.NOT_MATCHES}
   ```
   ```
   # オーバーライド
   Skip metadata collection for dynamic FS
   マクロ {#FSTYPE} : ^(btrfs|zfs)$
   ```
- アイテムのプロトタイプ「#FSNAME.getdata」を追加する
   依存アイテムとして、マスターアイテムに「vfs.fs.get」のエントリを指定
   データ型はテキスト、ヒストリは1h
   ```
   キー: vfs.fs.dependent[{#FSNAME},data]
   保存前処理: JSONPath $.[?(@.fsname=='{#FSNAME}')].first()
   ```
- アイテムのプロトタイプ「#FSNAME.pfree」を追加する(サンプル)
  依存アイテムとして、マスターアイテムに先ほどの「#FSNAME.getdata」のエントリを指定
  データ型はfloat
  ヒストリは31d、トレンドは365d…まあ適当でOK
  保存前処理は…公式テンプレートのそれを真似る
    JSONPath $.bytes.pfree()
- トリガのプロトタイプを追加する
  面倒なので「アイテムのプロトタイプ」を選択して「トリガーのプロトタイプを作成」でいいと思う