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

## VPSの構成

今回はVultrというサービスのサーバーを借りることにしました。
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

## ドメインとDNS

以前にCloudflare Registrarを利用して独自ドメインを取得していたので、今回はそのサブドメインを利用することにしました。
