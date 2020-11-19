# Raspberry Pi 4 Model B で遊ぶはなし（Raspberry Pi OS + Processing セットアップ編）

仕事の都合でRaspberry PiにProcessingを入れた環境が必要になったので、嬉々としてセットアップした記録です。ちなみにOSバージョンはこんな感じ。`hogehoge`の部分はホスト名。

```sh
% uname -a
Linux hogehoge 5.4.72-v7l+ #1356 SMP Thu Oct 22 13:57:51 BST 2020 armv7l GNU/Linux
```

## Raspberry Pi OSのインストール

[公式](https://www.raspberrypi.org/software/) からImagerをダウンロードして適当なPCにインストールしたら、Raspberry Pi OSを選んで、適当なSDカードに書き込むだけの超簡単作業。出来上がったSDカードをRaspberry Piに差して電源入れれば起動です。

## ザックリと時刻設定

起動すると勝手にセットアップウィザードが起動するので、適当に進めます。ただし、後々ツマラナイ理由でつまづくのが嫌なので、無線LANの設定辺りに来た時点で一度手を止め、ターミナルを起動して、時刻をザックリと合わせます。

```sh
% sudo date --set "yyyy/mm/dd HH:MM:SS"
```

終わったら、セットアップウィザードを継続、完走して再起動します。この時点で、インストール済みパッケージのアップデートまで終わっているはず。

## ちゃんと時刻設定

NTPの設定をするために`/etc/systemd/timesyncd.conf`を編集します。

```sh
% cat /etc/systemd/timesyncd.conf
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See timesyncd.conf(5) for details.

[Time]
NTP=ntp.jst.mfeed.ad.jp ntp.nict.jp
FallbackNTP=time.google.com
#RootDistanceMaxSec=5
#PollIntervalMinSec=32
#PollIntervalMaxSec=2048
```

先頭のNTPサーバは、特に事情がなければ、ネットワーク的に近いサーバを参照しておくべきです（実際の設定では、先頭にローカルNTPサーバを入れています）。設定ファイルを書いたので、NTPの有効化とサービスの再起動、起動確認です。

```sh
% sudo timedatectl set-ntp true
% sudo systemctl daemon-reload
% sudo systemctl restart systemd-timesyncd.service
% sudo systemctl status systemd-timesyncd.service
```

問題がなければ、最後のコマンドを実行すると`active (running)`という表示が見えたり、設定したNTPサーバとシンクロしている気配な何かが表示されるはず。

## JDKのインストール

Processingのインストールに先立って、JDKをインストールしておきます。

```sh
% sudo apt install default-jdk
```

## Processingのインストール

```sh
% curl https://processing.org/download/install-arm.sh | sudo sh
```

インストールが終わると、processingコマンドにパスが通っているはずなので、起動して動作確認します。無事に動けば、最低限の設定は完了。あとは便利に使うための細々とした設定です。

## ホスト名の設定

IPアドレスとか覚えていられないので、名前を付けます。

```sh
% sudo hostnamectl set-hostname <hostname>
```

これだけだと、DNSがこのホスト名を知らないネットワークに繋いでいると、名前解決ができないと怒られるので、`/etc/hosts`の`localhost`の部分を、上で設定した名前に書き換えておきます。

## IPアドレスのメール通知設定

今回は自分が管理する無線LANルータにぶら下げるのではなく、割と頻繁にパブリックなネットワークにDHCPで繋ぐ想定で環境を作っています。なので、ネットワークに繋がったらIPアドレスをGmail経由でメール通知してくれるように設定します。sSMTPでサクっと済ませたかったのに、どうもうまく行かない…似たような状態でハマっている人はたくさん見つかるのだけれども。というわけで、Exim4さんにお願いする。

### Gmail側の準備

事前準備として、Gmailの方で「安全性の低いアプリのアクセスを有効にする」というのを済ませておく必要があるようです。私は該当しなかったけど、２段階認証を使っている人は、アプリパスワードの取得が必要なのだとか。

### Exim4のセットアップ

というわけで、Exim4のインストール。なんか一緒にmailutilsも入った気配ですが、入らなかったら入れておくとテスト時などに便利です。

```sh
% sudo apt install exim4
```

Exim4の設定。今回はGmailに飛ばすだけ、受け取りは知らん、って感じなので、超絶適当。

```sh
% sudo dpkg-reconfigure exim4-config
```

* 最初の「メールサーバの設定」では「スマートホストでメール送信：ローカルメールなし」
* 「送出スマートホストのIPアドレスまたはホスト名」は「smtp.gmail.com:587」
* 「rootとpostmasterのメール受信者」はRaspberry Pi上の普段使っているユーザ
* そのほかの項目はデフォルトのまま

続いて、Googleさん向けの認証情報を`/etc/exim4/passwd.client`に書きます。伏字はメアドやパスワードです。

```sh
% sudo cat /etc/exim4/passwd.client
# password file used when the local exim is authenticating to a remote
# host as a client.
#
# see exim4_passwd_client(5) for more documentation
#
# Example:
### target.mail.server.example:login:password
gmail-smtp.l.google.com:xxxxxxxxxx@gmail.com:**********
*.google.com:xxxxxxxxxx@gmail.com:**********
smtp.gmail.com:xxxxxxxxxx@gmail.com:**********
```

設定を反映。

```sh
% sudo update-exim4.conf
```

ここらで`mail`コマンドで適当に動作確認をしておくと、後でトラブった時に問題の切り分けが楽。例えば、ここでの目標のように、無線LANのIPアドレスを調べてメールをするのであれば以下のようなコマンドで送信が可能。

```sh
% ip address show wlan0 | grep "inet " | mail -s "Your Raspberry Pi is now READY" xxxxxxxxxx@gmail.com
```

### メール送信スクリプト

せっかくなのでsystemdの流儀で追加してみます。まずは`/etc/systemd/system/autorun.service`の作成。

```sh
% cat /etc/systemd/system/autorun.service
[Unit]
Description=Execute at OS startup and terminates
After=network.target

[Service]
Type=oneshot
ExecStart=/home/pi/scripts/startup.sh
ExecStop=/home/pi/scripts/terminate.sh
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```

```sh
% ls -l /home/pi/scripts/
合計 4
-rwxr-xr-x 1 pi pi 140 11月 17 17:46 startup.sh
-rwxr-xr-x 1 pi pi   0 11月 17 17:43 terminate.sh
```

```sh
% cat /home/pi/scripts/startup.sh
#!/bin/sh

/sbin/ip address show wlan0 | /bin/grep "inet " | /usr/bin/mail -s "Your Raspberry Pi is now READY!" xxxxxxxxxx@gmail.com
```

サービスを有効化、テストします。

```sh
% sudo systemctl daemon-reload
% sudo systemctl enable autorun.service
% sudo systemctl start autorun.service
```

問題なければ、再起動して動作確認。

## SSHの有効化

`/boot`直下に空の`ssh`という名前のファイルがあれば良いそうなので、サクッと作って再起動します。細かい設定を弄りたい場合には、再起動前に`/etc/ssh/sshd_config`をイイ感じにしておきます。

## VNCの設定

`/etc/vnc/config.d/common.custom`に設定を追記。

```sh
% cat /etc/vnc/config.d/common.custom
#
# /etc/vnc/config.d/common.custom
#
Authentication=VncAuth
```

VNC用のパスワードを設定します。

```sh
% sudo vncpasswd -service
```

起動して動作確認。

```sh
% sudo systemctl start vncserver-x11-serviced.service
% sudo systemctl status vncserver-x11-serviced.service
```

ディスプレイが接続されていなくてもエラーを起こさないように、`/boot/config.txt`で`hdmi_force_hotplut=1`のコメントを外して、再起動します。

VNCクライアントソフト（Macなら画面共有.appなど）から`vnc://Raspberry PiのIPアドレス:5900`に繋ぐと、VNCで繋がります。

## 画面の解像度の変更

画面が狭いと何かと作業がしづらいので、いつもの解像度に変更します。コマンドラインから設定プログラムを起動します。

```sh
% sudo raspi-config
```

「2 Display Options」→「01 Resolution」と進んで、お好みの解像度に設定したら、指示された通りに再起動。

再びログインしたら「設定」→「Screen Configuration」→「Configure」→「Screens」→「HDMI-1」→「解像度」から、お好みの解像度に設定します。
