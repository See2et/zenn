---
title: "[home-manager] codexがError: Permission denied (os error 13)を吐く問題への対処法"
emoji: "📦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nix", "codex"]
published: true
---

## TL;DR
Nixの[home-manager](https://github.com/nix-community/home-manager)を用いたパッケージ管理環境へ移行していたところ、codexが正常に動作しませんでした。`.codex`を`home.file`で配置していたことにより、readonlyになっていたようです。codex は`~/.codex`に書き込みを必要とするため`Error: Permission denied (os error 13)`を吐いてしまいました。そのため、「必要なファイルだけを`home.file`で配置する」ことで対処しました。

## 検証環境
- Windows 11 Home
- WSL2
    - Ubuntu 24.04.3 LTS
- nix 2.31.2
- home-manager 25.11-pre
- codex-cli 0.42.0

## 起きていた問題

home-managerでは、宣言的にパッケージを管理することが出来ます。それに倣い、codexを以下のようにインストールしました。
```nix:home.nix
{config, pkgs}: {
    ...
    home.packages = pkgs; [
        neovim
        zsh
        ...
        codex
    ]
    ...
}
```
```zsh:zsh
// home.nixの変更を反映するコマンド
$ home-manager switch
```

しかし、codexを正常に動作させることが出来ませんでした。
```zsh:zsh
$ codex
Error: Permission denied (os error 13)
```

## 問題の切り分け
まず、`codex`コマンドそれ自体が正常に動作していないのか、それ以外のread/writeが必要な場面で詰まっているのかを切り分けます。幸か不幸か`codex --help`は正常に動作しました。
```zsh:zsh
$ codex --help
Codex CLI

If no subcommand is specified, options will be forwarded to the interactive CLI.

Usage: codex [OPTIONS] [PROMPT]
       codex [OPTIONS] [PROMPT] <COMMAND>
...
```

codexは`~/.codex`配下にログ等を読み書きするので、ここの権限不足を疑います。どうやら、書き込みが禁止されていることが分かります。

```zsh:zsh
$ cd ~/.codex
$ lsd -la
dr-xr-xr-x   2 see2et see2et 4.0K Jan  1  1970 .
drwxr-xr-x 517 see2et see2et 356K Oct  1 04:53 ..
-r--r--r--   1 see2et see2et  145 Jan  1  1970 AGENTS.md
-r--r--r--   1 see2et see2et  504 Jan  1  1970 config.toml
-r--r--r--   1 see2et see2et  395 Jan  1  1970 github-mcp.sh
```

`chmod 700 .`とかで権限を与えてもいいのですが......これは宣言的になるべく全てを扱いたいNixの思想に反するので、お作法的に避けたいところです。なので、根本的な原因を解決することにしました。

## 原因の正体
`~/.codex`がreadonlyになっていた原因は、以下のようにcodexの設定ファイルを管理していたことにありました。この設定により、`home-manager switch`実行時に`~/.config/home-manager/codex`は`~/.codex`に自動で配置されます。
:::details なぜ`~/.config/home-manager`にcodexの設定ファイルを配置するのか
dotfilesをgitやchezmoiでホーム直下に直接配置してしまうと、その場では動きますが「どのファイルをどう管理しているか」が曖昧になってしまいます。home-managerを使うことで、Nix の設定ファイルに「どの dotfile をどこに置くか」をすべて宣言的に書けるようになります。

これにより、Git で管理している自分の設定リポジトリを Nix 設定にひもづけておけば、別のマシンでも home-manager switch を叩くだけで同じ環境を一発で再現することが出来ます。そのため、直接 ~/.codex にファイルを置くのではなく、いったん ~/.config/home-manager/codex にソースをまとめ、Home Manager の home.file オプション経由でホームディレクトリに展開する方式を取っていました。
:::
```nix:home.nix
{config, pkgs}: {
    home.file = {
        ".config/nvim".source = ./nvim;
        ".config/zellij".source = ./zellij;
        ...
        ".codex".source = ./codex;
    };
}
```
このときhome-managerは`/nix/store/`配下のビルド済ファイルへのリンクを配置するため、readonlyとして扱われてしまうようです。そこで、必要なファイルだけをhome-managerで配置することで、この問題を解決しました。
```diff:home.nix
{config, pkgs}: {
    home.file = {
        ".config/nvim".source = ./nvim;
        ".config/zellij".source = ./zellij;
        ...
-   ".codex".source = ./codex;

+   # 必要なファイルだけを配る（親ディレクトリは実体として作られる）
+   ".codex/config.toml".source = ./codex/config.toml;
+   ".codex/AGENTS.md".source = ./codex/AGENTS.md;
+   ".codex/github-mcp.sh" = {
+       source = ./codex/github-mcp.sh;
+       executable = true;
+   };
}
```

無事、`$ codex`で立ち上げることができました。確認してみると、きちんとwrite権限が付与されていますね。`sessions`,`log`などのフォルダも正常に作成されていることが分かります。

```zsh:zsh
$ lsd -la
drwx------ see2et see2et 4.0 KB Wed Oct  1 16:49:10 2025  .
drwxr-x--- see2et see2et 4.0 KB Wed Oct  1 16:54:53 2025  ..
lrwxrwxrwx see2et see2et  79 B  Wed Oct  1 16:43:45 2025  AGENTS.md ⇒ /nix/store/z34866j787zlxab15w7b5sima0wcr7ld-home-manager-files/.codex/AGENTS.md
lrwxrwxrwx see2et see2et  81 B  Wed Oct  1 16:43:45 2025  config.toml ⇒ /nix/store/z34866j787zlxab15w7b5sima0wcr7ld-home-manager-files/.codex/config.toml
lrwxrwxrwx see2et see2et  83 B  Wed Oct  1 16:43:45 2025  github-mcp.sh ⇒ /nix/store/z34866j787zlxab15w7b5sima0wcr7ld-home-manager-files/.codex/github-mcp.sh
drwxr-xr-x see2et see2et 4.0 KB Wed Oct  1 16:48:41 2025  log
drwxr-xr-x see2et see2et 4.0 KB Wed Oct  1 16:49:10 2025  sessions
.rw-r--r-- see2et see2et  79 B  Wed Oct  1 16:48:42 2025  version.json
```

## 最後に
ここまでご一読下さり、ありがとうございました。良きhome-managerライフの一助となりましたら幸いです。
