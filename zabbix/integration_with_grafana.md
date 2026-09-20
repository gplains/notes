
---
title:Grafanaとの連携
---
# Grafanaと連携する

## 目的

- なんかGrafana入れたくなっちゃって

## 参考にしたURI

### まずGrafanaのインストール

RHELの場合。雑に grafana.repoを流し込む

```
sudo cat << 'EOF' > /etc/yum.repos.d/grafana.repo
[grafana]
name=grafana
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
EOF
```

とりあえず 3000/tcpでプロセスが立つところまで
```
dnf install grafana
sudo dnf install -y grafana
sudo systemctl daemon-reload
sudo systemctl enable --now grafana-server
sudo firewall-cmd --add-port=3000/tcp --permanent
sudo firewall-cmd --reload
```

grafanaのwebuiの初期パスワードはadmin/adminだった気がする、あとは自分で書き換えて

### zabbix連携プラグインのインストール

一部の参考サイトでは「grafana-cliを使え」とあったが、今は grafana cli らしい
```
sudo grafana cli plugins install alexanderzobnin-zabbix-app
sudo systemctl restart grafana-server
```

事前にzabbixでgrafana 連携用アカウントを作っておく、ロールは...なんだったかな

### 見た目同じポートにしたい場合

mod_proxy モジュールを使ってアレする

```
sudo vi  /etc/grafana/grafana.ini
# [server]
# domain = host.mydom.local
# root_url = %(protocol)s://%(domain)s:%(http_port)s/grafana/
# serve_from_sub_path = true

sudo vi /etc/httpd/conf.d/grafana.conf
# <Location /grafana>
#     ProxyPass http://localhost:3000
#     ProxyPassReverse http://localhost:3000
# </Location>

sudo systemctl restart grafana-server
sudo apachectl configtest
sudo systemctl reload httpd
```





