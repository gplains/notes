
---
title:make_local_rpm_repository
---
# ローカルネットワークに簡易的なRPMリポジトリを作る

- 想定されるユースケース
  
  特定の構成のVMを作っては怖しする場合(dnf install はするがdnf updateまではしない)

  CD-ROM(DVD-ISO)の挿抜が大変なので簡略化したい


## nginxで公開パスを定義する

この場合、 http://hostip/repos にアクセスすると /mnt/repos を参照する感じね

```
location /repos {
root /mnt ;
}
```

## DVD ISOイメージをフォルダに配置する

```
# 対象ISOをループバックでマウント
sudo mkdir /mnt/cdrom
ls -l /datastore/*.iso # 例えば、rhel-8.10-x86_64-dvd.iso   がある
mount -o loop /datastore/rhel-8.10-x86_64-dvd.iso  /mnt/cdrom
 
# コピー先フォルダを切って配置
sudo mkdir -p /mnt/repos/rhel8a  #8.10にしてもいいんだけどなんか見栄えがアレなので
cd /mnt/cdrom
cp -r ./ /mnt/repos/rhel8a    # 少し時間かかるのでマウンテンクライマーしながら待機
cd /mnt
umount /mnt/cdrom
ls -l /mnt/repos/rhel8a        # TRANS.TBLやBaseOS/AppStreamが見えればOK
```

## 実際にRHEL VMで設定する項目...

```
sudo vi /etc/yum.repos.de/local.repo
 
[rhel8a_BaseOS]
name=RHEL8.10 BaseOS
baseurl=http://hostip/yum/rhel8a/BaseOS
enabled=1
gpgcheck=0
[rhel8a_AppStream]
name=RHEL8.10 AppStream
baseurl=http://hostip/yum/rhel8a/AppStream
enabled=1
gpgcheck=0
```

## 特定のRPMを持ってきて私家版リポジトリを作る場合

想定されるレイアウト
```
/mnt/repos
      |
      `-- private
           |
           `-- Packages
```

実際

```
sudo dnf install createrepo # createrepo_c createrepo_c-libs drpm
sudo mkdir -p /mnt/repos/private/Packages
# 適当な方法で /mnt/repos/private/Packages にrpm を配置
createrepo /mnt/repos/private/

# 以下、ウェブサーバが非RHELだった場合
cd /mnt/repos
tar cvf private.tar private  # 生成したprivate.tar を転送
# 受信したウェブサーバで tar xvf private.tar で展開...
```

先述の local.repo に以下を追記する

```
[private]
name=Private Repo
baseurl=http://hostip/yum/private
enabled=1
gpgcheck=0
```

## 参考URL

- https://runningdog.mond.jp/blog/2023/03/16/almalinux%E3%81%AE%E3%83%AD%E3%83%BC%E3%82%AB%E3%83%AB%E3%83%AC%E3%83%9D%E3%82%B8%E3%83%88%E3%83%AA%E3%82%92%E6%A7%8B%E7%AF%89%E3%81%99%E3%82%8B%E3%82%88/