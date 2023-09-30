---
title: "Misskeyのおひとり様インスタンスを建てる"
emoji: "🌱"
type: "tech"
topics: [misskey]
published: false
---

# 初めに

先日旧TwitterことXにて、私の投稿が他の人から見えないいわゆるシャドウバン状態になってしまいました。
その普及率の高さゆえにXから逃れることは難しいですが、安心と安全のために自分の管理下にあるSNSも欲しいな...と思い立ちMisskeyインスタンスを建てることにしました。
元々Misskey.ioのアカウントは持っていたのですが、どうせ分散型SNSを利用するなら自分のインスタンスを持ってリレーに参加したい...という願望があったのも一因です。
当記事は今回作った[しぜすきー](https://mi.see2et.dev)の構築の様子の備忘録です。

## 用意したもの

- Misskeyを実行するためのVPS: Vultr
- 独自ドメイン: Cloudflare Registrar
- DNS: Cloudflare
- オブジェクトストレージ: Cloudflare R2

### VPSの構成

今回はVultrというサービスのサーバを借りることにしました。
スペックは下記の通りです。

| 項目 | 内容 |
| ---- | ---- |
| Server | Cloud Compute |
| Region | Tokyo |
| CPU | 1 |
| RAM | 1024MB |
| Storage | 25GB |
| OS | Ubuntu 22.04 x64 |

デフォルトで付与されているAuto Backupsオプションをオフにすれば、月額5ドルで利用できるみたいです。(2023年9月30日現在)

### ドメインとDNS

Fediverseにおいてドメインは@example@example.comのようにユーザ名の一部になるため、独自ドメインを使用することをお勧めします。
以前にCloudflare Registrarを利用してドメインを取得していたので、今回はそのサブドメインを利用することにしました。
連携が便利で情報が豊富なため、DNSもCloudflareのものを利用しました。

### オブジェクトストレージ

「10GB/月まで無料」「データのアクセスにお金はかからないぜ！」というお得な料金体系であるCloudflare R2を利用します。
オブジェクトストレージを利用しなくてもMisskeyインスタンスを運用することができますが、画像やカスタム絵文字を利用することはできません。

## ssh接続と環境構築

### ssh接続の準備

まずはあなたのコンピュータでssh鍵を作成しましょう！


```$ ssh-keygen -t ed25519```


これで秘密鍵が`~/.ssh/id_ed25519`、公開鍵が`~/.ssh/id_ed25519.pub`に生成されました。

次に`.ssh/config`を編集します。

```
Host misskey
    User <お好きな名前をば。今回はsee2etで進めていきます。>
    Port 22
    Hostname <VPSのIPアドレス>
    IdentityFile ~/.ssh/id_ed25519
```

### sshでサーバに接続

以下の内容を実行してVPSサーバにアクセスします。

```$ ssh root@misskey -p 22```

rootのパスを確認されるので、VPSに記載されているものをコピペしましょう。

もし`WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`と怒られてしまったら、以下の記事を参考にすると良いかもしれません。
[「SSHホスト鍵が変わってるよ！」と怒られたときの対処](https://qiita.com/hnw/items/0eeee62ce403b8d6a23c)

### 新規ユーザの作成

どうやらrootでssh接続するのはよろしくないみたいなので、作業用のユーザを作成します。

```$ adduser see2et```

該当ユーザでsudoを扱えるようにします。

```$ usermod -aG sudo see2et```

### 公開鍵・秘密鍵利用のセットアップ

公開鍵・秘密鍵を利用してssh接続できるようにします。
ファイル編集にviを利用していますが、適宜nanoなどを使用しても構いません。

```
$ sudo -iu see2et
$ mkdir .ssh
$ chmod 700 .ssh
$ cd .ssh
$ touch authorized_keys
$ chmod 600 authorized_keys
$ vi authorized_keys
```

authorized_keysに、以下のコマンドを**ssh接続元のコンピュータ**で実行して確認できた公開鍵をコピペします。

```
$ cat .ssh/id_ed25519.pub 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIP6n+UaBX1typlu2xQAlmrISPMyIB7VO1MbuzUz0gpPL see2et@see2et-mbp.local
```

次に、sshdの設定を変更します。

```$ sudo vi /etc/ssh/sshd_config```

以下の設定を有効化しましょう。必要に応じて行頭の#を削除してください。

- PermitRootLogin no
- PasswordAuthentication no

ここまでできたらsshdの設定をテストし、エラーがなければ有効化のためにsshdを再起動しましょう。

```
$ sudo sshd -t
$ sudo systemctl restart sshd
```

### ファイアウォール

utfを利用して特定のポートの接続だけを許可します。
必ず`.ssh/config`に記述したものを利用しましょう。

```
$ sudo utf allow 22/tcp
$ sudo utf enable
```
