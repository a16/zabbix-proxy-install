# 通常インストール(Centos7)

1. SELinux無効化  
/etc/selinux/configを編集し、SELINUX=`enfocing`から`disabled`に変更


```bash:/etc/selinux/config
% sudo vi /etc/selinux/config
- SELINUX=enfocing
+ SELINUX=disabled
```

2. yum update

```bash
% sudo yum update
```

3. 再起動

```bash
% reboot
```

4. zabbix3.4のリポジトリ登録

```bash
% sudo rpm -ivh http://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-release-3.4-1.el7.centos.noarch.rpm
```

5. パッケージインストール

```bash
% sudo yum install zabbix-proxy-sqlite3
yes/noを聞かれたらy
```

6. sqlite3スキーマ読み込み

```bash
% sudo mkdir /var/lib/zabbix
% sudo chown zabbix:zabbix /var/lib/zabbix
% sudo -u zabbix sh -c "zcat /usr/share/doc/zabbix-proxy-sqlite3-3.4.14/schema.sql.gz | sqlite3 /var/lib/zabbix/zabbix_proxy.db"
```

7. zabbix-proxy config設定  
拠点ごとに違うので別途情報お渡しします。

```bash
% sudo vi /etc/zabbix/zabbix_proxy.conf
※渡したファイルの内容をコピペするかパラメータを修正してください。

% sudo vi /etc/zabbix/**********.psk
※渡したファイルの内容をコピペしてください。

% sudo chown -R zabbix:zabbix /etc/zabbix
```

以下は内容の説明

```bash
% sudo cat /etc/zabbix/zabbix_proxy.conf
ProxyMode=1  # Passiveモード
Server=x.x.x.x # zabbix-serverのIPアドレス
Hostname=Proxy_y.y.y.y # zabbix-proxyのホスト名。zabbix-serverとzabbix-proxy間でuniqなものを設定する必要がある。Proxy_(zabbix-proxyのIPアドレス)とします。
SourceIP=y.y.y.y # zabbix-proxyのIPアドレス
LogFileSize=1 # logのファイルサイズを1MBまで
PidFile=/var/run/zabbix/zabbix_proxy.pid # PIDファイル
LogFile=/var/log/zabbix/zabbix_proxy.log # logファイル
DBName=/var/lib/zabbix/zabbix_proxy.db # sqlite3のDBファイル
CacheSize=16M # デフォルトは8Mだが、監視対象が3,000くらいになるとエラーが出るので増やしておく。検証では監視対象4,000までは大丈夫だった。
StartHTTPPollers=4 # web監視用の起動ワーカー数。デフォルトは1。監視対象が3,000くらいになるとhttp poller proccessのアラームが出るので4に増やしておく。
StartTrappers=1 # zabbix trapperのワーカー数。デフォルトは5。とりあえず使わないので1にしておく。
TLSConnect=psk # zabbix-proxyからzabbix-serverへの接続をSSL暗号化する方法の指定(事前公開鍵利用)
TLSAccept=psk # zabbix-serverからzabbix-proxyへの接続をSSL暗号化する方法の指定(事前公開鍵利用)
TLSPSKIdentity=******** # PSKのid。zabbix-serverとzabbix-proxy間でuniqなものを設定する必要がある。
TLSPSKFile=/etc/zabbix/**********.psk # PSKの配置場所
```

8. zabbix-proxy起動

```
% sudo systemctl enable zabbix-proxy
% sudo systemctl start zabbix-proxy
```

インストール完了。
