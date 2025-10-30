
---
title:qcow2_virt-customize
---
# RHELのクラウドイメージにvirt-customizeで管理ユーザを追加する

- ダウンロード直後のクラウドイメージの制限

  rootアカウントがロックされている、パスワード再設定が必要

  そもそもrootアカウントでSSH接続できない

  ほかのユーザが存在しない、コンソール接続して追加しないといけない

## virt-customize での加工例

- 前準備:スナップショットを採取する

  qemu-img snapshot -c default  rhel8a_base.qcow2

- rootアカウントのパスワードを再設定する(root-password を使用)

  virt-customize -a rhel8a_base.qcow2 --root-password password:hogehoge

- 新規にユーザを追加する(useraddとchpasswdをワンライナーで流す)

  virt-customize -a rhel8a_base.qcow2 --firstboot-command \

  'useradd -G wheel user ; echo user:password | chpasswd ' -v -x

- 後始末:スナップショットを破棄する(シャットダウン後で)

  qemu-img snapshot -d default  rhel8a_base.qcow2

## cloud-init での加工例

- qcow2_initial.md の項を参照...