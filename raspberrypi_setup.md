# Raspberry Pi 4 Model B で遊ぶはなし（セットアップ編）

## Ubuntu のインストール

大変素晴らしく、トラブルは一切なし。[公式](https://ubuntu.com/download/raspberry-pi)からダウンロード、SSDに書き込むだけです。今回はUbuntu Server 19.10 64-bitを入れました。OSインストール大好きなので。デフォルトユーザでログイン、パスワードを変更しておきます。

（後日追記）
新しいバージョンがリリースされたのでアップグレードします。新しいもの好きなので。ただしパッチをひと通り当ててリブートしてから。

```sh
% sudo apt update
% sudo apt upgrade
% sudo shutdown -r now
% sudo do-release-upgrade
```

## 細かな設定いろいろ

### タイムゾーンの変更

ログ読む時に困るので、とりあえずタイムゾーンを変更。

```sh
% sudo unlink /etc/localtime
% sudo ln -s /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
```

### デフォルトエディタの変更

古い人間なので、設定ファイル編集する際に見知らぬエディタが上がってくるとドキドキしてしまうので、環境変数で知っているエディタが上がってくるようにしておきます。

```sh
% sudo update-alternatives --config editor
```

この辺りまで設定したら、一度リブート。

### ホスト名の変更

手元にいくつかラズパイやSSDがあると判別が面倒なので、適当に名前をつけておきます。

```sh
% sudo hostnamectl set-hostname <hostname>
```

## 無線LANの設定

### 状況確認

```sh
% uname -a
Linux ubuntu 5.3.0-1014-raspi2 #16-Ubuntu SMP Tue Nov 26 11:18:23 UTC 2019 aarch64 aarch64 aarch64 GNU/Linux

% lspci -k
00:00.0 PCI bridge: Broadcom Inc. and subsidiaries Device 2711 (rev 10)
    Kernel driver in use: pcieport
01:00.0 USB controller: VIA Technologies, Inc. VL805 USB 3.0 Host Controller (rev 01)
    Subsystem: VIA Technologies, Inc. VL805 USB 3.0 Host Controller
    Kernel driver in use: xhci_hcd

% ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
    link/ether dc:a6:32:71:ad:d9 brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether dc:a6:32:71:ad:da brd ff:ff:ff:ff:ff:ff

% sudo ip link set wlan0 up
```

とりあえず、何も怒られないところをみると、無線LANデバイスそのものは問題なく動いている模様。

### WPA2 Personalな無線LANの設定

まずは以下の状況の無線LANに接続をしてみます。ルータでは予め、Raspberry Piの無線LANモジュールのMACアドレスを登録して、接続を許可しておきます。

- ステルスなSSID
- 認証方式はWPA2-Personal
- WPAの暗号化方式はAES
- WPA-PSKキーあり
- DHCPでIPアドレスを取得

まずはwpa_supplicantの設定。

```sh
% sudo wpa_passphrase <SSID> <パスフレーズ> | sudo tee -a /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
```

`wpa_supplicant.conf`のマニュアルを見ながら、フロントエンド向けグループ設定、ステルス探す設定、認証方式、暗号化方式を追記して、コメントアウトされている平文パスワードを削除します。

```sh
# allow frontend (wpa_cli etc) to be used by all users in sudo group
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=sudo

network={
    ssid="SSIDがここに"
    scan_ssid=1
    key_mgmt=WPA-PSK
    psk=<暗号化されたWPA-PSKキーがここに>
    pairwise=CCMP
    group=CCMP
}
```

保存したら、念のため今走っているwpa_supplicantに向かって`killall`して新しい設定ファイルで走らせてみます。

```sh
% sudo killall wpa_supplicant
% sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
Successfully initialized wpa_supplicant
```

`-B`オプションはバックグラウンド実行です。うまくいったら、DHCPでIPアドレスを取得してみます。

```sh
% sudo dhclient wlan0
% ip address show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether dc:a6:32:71:ad:d9 brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether dc:a6:32:71:ad:da brd ff:ff:ff:ff:ff:ff
    inet 192.168.50.185/24 brd 192.168.50.255 scope global dynamic wlan0
       valid_lft 86235sec preferred_lft 86235sec
    inet6 fe80::dea6:32ff:fe71:adda/64 scope link
       valid_lft forever preferred_lft forever
```

ひと通りの動作確認ができたので、起動時に無線LANに接続するように設定します。まずはsystemd-networkdの設定（`/etc/systemd/network/10-wlan0-dhcp.network`）。

```sh
[Match]
Name=wlan0

[Network]
DHCP=ipv4

[DHCP]
UseRoutes=true
```

これで起動時にwlan0に火が入るので、wpa_supplicantをsystemctlに登録。

```sh
% systemctl enable wpa_supplicant@wlan0
```

再起動して、IPアドレスを取って来ていることを確認します。ただ、起動時に、NICの設定が終わるまで、やたらと待つことに気がつきます。

```sh
% systemctl --failed
% systemctl status systemd-networkd-wait-online.service
```

systemd-networkd-wait-online.serviceがコケてる。

```sh
% systemctl show -p WantedBy network-online.target
WantedBy=iscsid.service cloud-config.service open-iscsi.service cloud-final.service
```

うーん…殺しても大丈夫そうなサービスが混ざっている気がするけど、少し怖いので保留。ぐぐると多くの人が同様の問題で困っているけど、綺麗な解決法が見つからない。

ちなみに、どのサービス起動にどのくらいの時間がかかっているのかは、コマンドで確認可能。

```sh
% systemd-analyze blame
% systemd-analyze critical-chain
```

### IEEE802.1Xな無線LANの設定

今度は、802.1Xで認証する無線LANに繋ぐ設定をします。

- ステルスじゃないSSID
- 認証方式はWPA2-Enterprise
- WPAの暗号化方式はAES（パスフレーズなし）
- IEEE802.1X (EAP-PEAP)でユーザ認証
- DHCPでIPアドレスを取得

`/etc/wpa_supplicant/wpa_supplicant-wlan0.conf`にnetworkのブロックを複数書いておくと、繋がるものに繋げに行くそうで。そこでnetworkブロックをひとつ追加。

```sh
network={
    ssid="SSIDがここに"
    key_mgmt=WPA-EAP IEEE8021X
    pairwise=CCMP
    group=CCMP
    eap=PEAP
    identity="IEEE802.1X認証のユーザIDがここに"
    password="IEEE802.1X認証のパスワード（平文！）がここに"
    phase1="peaplabel=0"
}
```

これ、「一番良さげなものに繋げに行く」ではなく、複数のプロファイルを切り替えて使うことってできないんだろうか。あとsyslog読むといろいろ気持ち悪いけど、闇が深そうなので見なかったことに。

## パッケージのアップデート

いつものアレ。

```sh
% sudo apt update
% sudo apt full-upgrade
```
