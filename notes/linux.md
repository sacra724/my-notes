# linux

## よく使うコマンド

### 時刻

```
$ systemctl restart network

$ ntpdate -v time.google.com
```



* [git]
* [NodeJs関連]
* [MySQL]
* PHP関連
	* [PHP]
* [Nginx]
* [zsh関連]

## VirtualBoxの設定 (ローカル環境用メモ)

### 新規作成

* [Centos7をDL](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1503-01.iso)
*「新規」クリック
	* 名前 : Centos7
	* タイプ : Linux
	* バージョン : RedHat(64bit)
* メモリはとりあえず「1024MB」
* 「VDI」選択
* 「固定」選択。「固定」と「可変」はどっちでもいいと思う
* HDの容量はなんとなく30GBくらいにしておこうと思う


### ネットワーク

* 「VirtualBox」->「環境設定」->「ネットワーク」->「ホストオンリーネットワーク」
* 「+」アイコンクリック
	* 「ドライバーみたいなアイコン」クリック
		* アダプターは初期設定のまま
			* 2台目以降を繋ぐ場合はこれを使い回す
		* DHCPサーバー「サーバーを有効化」チェック
			* サーバーアドレス : 192.168.56.100
			* サーバーマスク : 255.255.255.0
			* アドレス下限 : 192.168.56.101
			* アドレス上限 : 192.168.56.254
* 作成した環境を選択し、「設定」クリック
	* 「ネットワーク」->「アダプター１」->「割り当て」-> ブリッジアダプター (自分の環境だと en0 Wi-Fi)
	* 「アダプター２」->「割り当て」-> ホストオンリーアダプター (上記で作成)


### 起動

* 作成した環境を選択して「起動」クリック
* フォルダーアイコンクリック
	* DLしたCentos7のisoを選択
* 先程設定したHDDにインストール
* passwordを設定する

## 設定

### 参考サイト

* [さくらVPSにCentOS7入れてみたログ](http://qiita.com/dansuke@github/items/4cb01478d135b706c8fd)
* [VirtualBox に CentOS7.0 をインストールして ssh ログイン + yum コマンドを使えるようにするまで](http://d.hatena.ne.jp/shouh/20150429/1430283666)
* [さくらのVPSを使うときに最初にやっておきたいこと(CentOS 7編)](http://server-setting.info/blog/sakura_vps_centos7_first_setup.html)
* [ifconfig　～（IP）ネットワーク環境の確認／設定を行う](http://www.atmarkit.co.jp/ait/articles/0109/29/news004.html)
* [リポジトリを追加する](http://www.server-world.info/query?os=CentOS_7&p=initial_conf&f=6)

ネットワークの設定が難しすぎる

```bash
$ vi /etc/pam.d/su
# 以下の行のコメントを外す
auth required pam_wheel.so use_uid


$ visudo
# 以下の行のコメントを外す
%wheel ALL=(ALL) ALL

# 新規ユーザーを作成し、wheelに所属させる
$ useradd <NEW_USER_NAME>
$ passwd <YOUR_PASSPHRASE>
$ usermod -G wheel <NEW_USER_NAME>


# SELinuxをOffに
# 動作状況
$ getenforce
Enforcing

# 一時的に無効に
$ setenforce 0

$ getenforce
Permissive

$ vi /etc/selinux/config
SELINUX=enforcing
↓　変更
SELINUX=disabled

$ sudo vi /etc/ssh/sshd_config
#ポート設定
Port 22

#ルートログイン設定
#PermitRootLogin no
PermitRootLogin yes

#鍵認証設定
#RSAAuthentication yes
#PubkeyAuthentication yes

#パスワードでのログイン
PasswordAuthentication yes


#ファイアーウォールの設定
$ systemctl status firewalld

$ firewall-cmd --add-service=http    ←使用するサービスにhttpを追加
success

$ firewall-cmd --permanent --add-service=http    ←再起動してもhttpを使用
success

$ firewall-cmd --add-port=22/tcp    ←22ポートを追加
success

$ firewall-cmd --permanent --add-port=22/tcp    ←再起動しても22を使用
success

$ firewall-cmd --list-services     ←サービスを確認（httpが追加されてる）
dhcpv6-client http ssh

$ firewall-cmd --remove-service=ssh    sshを削除（sshを消す意味がわからないけど、皆さんやってるので。理由は元気があったら調べる）
dhcpv6-client http

$ firewall-cmd --list-ports    ←ポートの確認（22が追加されてる）
22/tcp


# Centos7のネットワーク設定がすごく難しいので注意
# 自分はホストオンリーの設定が上手くいかなかった
# ifcfg-enp0s8 で IPADDRとGATEWAYを設定したら繋がった
$ nmtui

もしくは

$ cd /etc/sysconfig/network-scripts
$ vi ifcfg-enp0s3 (ブリッジアダプター)

☆ = 追加・変更したもの

HWADDR=01:23:45:AB:CD:E0 ☆
TYPE=Ethernet
BOOTPROTO=dhcp ☆
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no ☆
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=enp0s3
UUID=123123123-12123-123123-123123-123123123
DEVICE=enp0s3
ONBOOT=yes ☆
PEERDNS=yes
PEERROUTES=yes

$ vi ifcfg-enp0s8 (ホストオンリーアダプター)

HWADDR=01:23:45:AB:CD:E0 ☆
TYPE=Ethernet
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no ☆
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=enp0s8
UUID=123123123-12123-123123-123123-123123123
DEVICE=enp0s8
ONBOOT=yes ☆
IPADDR=192.168.56.101 ☆
PREFIX=32

## ここから先はitermで作業

$ ssh <NEW_USER_NAME>@192.168.56.101

## yum で色々入れる前の準備

$ sudo yum -y update　←　インストール済パッケージの一括アップデート
※大量のパッケージのダウンロード／アップデートを行うため時間がかかる
*つながらない時 [VirtualBoxのCentOS7(CUI)でブリッジ接続でインターネットに繋がらない時の対応](https://goo.gl/drnQRm)

$ sudo yum -y install yum-cron　←　yum-cronインストール

$ sudo vi /etc/yum/yum-cron.conf　←　yum-cron設定
# Whether updates should be applied when they are available.  Note
# that download_updates must also be yes for the update to be applied.
apply_updates = yes　←　ダウンロード&アップデートを自動で行うようにする

$ sudo systemctl start yum-cron　←　パッケージ自動更新起動

$ sudo systemctl enable yum-cron　←　パッケージ自動更新自動起動設定

$ sudo yum -y groupinstall base "Development tools"　←　ベース、開発ツールパッケージ群インストール
```
