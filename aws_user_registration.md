# AWSへのユーザ登録

事情により同じ作業を毎年行うのですが、すぐに忘れてしまうトリアタマなので
忘備録です。

## EC2インスタンスのセットアップ

今回はAmazon Linux 2 AMI (HVM), SSD Volume Typeを使ってみます。
特に拘りの理由はないです。Linuxなら何でもよくて、一番上に
あったからと言う安直な理由です。

インスタンスタイプは、t2.microに。
本当はもう少しスペックが良いインスタンスを使おうかと思ったのですが、
MySQLを立てた後にSQL文の投げ方の違いによるパフォーマンスの差などを
見る実験をお手軽にしたかったので、程よくt2.microにしました。

とりあえず残りもデフォルト設定のままで立てて、キーペアを作成、
ダウンロードして、インスタンスを作成します。

## ログインと初期セットアップ

先ほどダウンロードした鍵のパーミッションを0400に変更して、
デフォルトのユーザ(ec2-user)でログインします。

参考：Linuxインスタンスでのユーザーアカウントの管理
<https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/managing-users.html>

念のためパッケージのアップデート。

```sh
% sudo yum update
```

タイムゾーンを変更します。以下のコマンドで存在を確認後、`ZONE="Japan"`に変更。
さらにシンボリックリンクを作成。再起動。

```sh
% ls /usr/share/zoneinfo/
% sudo vi /etc/sysconfig/clock
% sudo ln -sf /usr/share/zoneinfo/Japan /etc/localtime
% sudo reboot
```

参考: Linuxインスタンスの時刻の設定
<https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/set-time.html>

## ユーザアカウントの作成と鍵の登録

参考： Linuxインスタンスでのユーザーアカウントの管理
<https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/managing-users.html>

ローカルで`ssh-keygen`を実行して予め鍵を作っておき、公開鍵をscpしておく。`/tmp`などに置いておくと後の作業が楽になるが、作業が終わったら必ず消しておく。

ユーザを作成して鍵を`authorized_keys`にコピー。例えば新規ユーザ名を`hogehoge`とするならば、以下の通り。

```sh
% sudo adduser hogehoge
% sudo su - hogehoge
% mkdir .ssh
% chmod 0700 .ssh
% touch .ssh/authorized_keys
% chmod 0600 .ssh/authorized_keys
% cat /tmp/hogehoge.pem.pub > .ssh/authorized_keys
```

たくさんユーザ登録をする時には、スクリプトを書いて処理してます。

```sh
#!/bin/bash

if [ $# -eq 2 ]; then
    files=$1/*
    names=(`cat "$2"`)
else
    echo "Search files in the current directory."
    files="./*"
    names=
fi

ln=0
for filepath in ${files}; do
    if [ -n "${names}" ]; then
        name=${names[${ln}]}
        `sudo adduser -m ${name}`
        `sudo mkdir -p /home/${name}/.ssh`
        `sudo touch /home/${name}/.ssh/authorized_keys`
        `sudo chmod 0600 /home/${name}/.ssh/authorized_keys`
        `cat ${filepath} | sudo tee -a /home/${name}/.ssh/authorized_keys`
        `sudo chown -R ${name}.${name} /home/${name}/.ssh`
        let ln++
    fi
done

exit 0
```

スクリプト実行時、第1引数に鍵ファイルを置いてあるディレクトリ名、第2引数にユーザ名一覧が書いてあるファイル名を指定して使います。
