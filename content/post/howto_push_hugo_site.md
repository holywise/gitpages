+++
date = "2017-01-15T21:26:28+09:00"
tags = ["Windows","Hugo", "GitHub"]
isCJKLanguage = true
slug = "how_to_push_hugo_site"
title = "HUGOで静的ページサイトを構築する (3/3)"
draft = false
categories = ["環境構築"]
author = "holywise"
description = "HUGOで静的ページを作成して GitHub Pages で公開するまでの手順"

+++

## 構築作業の全景

この一連のポストでは以下の手順で進めていくことになります。

1. [x] HUGOをインストールする
1. [x] プロジェクト用の作業ディレクトリを作る
1. [x] 作業ディレクトリに各種の初期設定を行う
1. [X] 作業ディレクトリにページデータを作成する
1. [ ] **GitHub Pages に連携する** (←)

[前回のポスト]({{< relref "howto_add_hugo_pages.md" >}})で4番目の手順まで終えました。
当ポストでは5番目の手順を示します。

<!--more-->

## GitHub Pages のリポジトリを作成する

今回の例のように 個人ブログを GitHub Pages でページを公開したい場合は、
`yourname.github.io` という名称のリポジトリを作成する必要があります。
`yourname` の部分はユーザーのアカウント名が入ります。
筆者の例なら `holywise.github.io` となります。

リポジトリの具体的な作り方自体は[いろんな方々が方法を紹介](https://www.google.co.jp/search?q=github+%E3%83%AA%E3%83%9D%E3%82%B8%E3%83%88%E3%83%AA+%E4%BD%9C%E6%88%90&ie=utf-8&oe=utf-8)されているので、
そちらを参照してください。

[GitHub Help](https://help.github.com/articles/user-organization-and-project-pages/)
に記載のあるとおり、この `yourname.github.io` リポジトリの `master` ブランチが公開対象となります。

## 公開用ページデータのビルドを試す

### 公開するページのステータスを変更する

まずは公開対象としたいページ原稿データそれぞれについて、ステータスを公開にする必要があります。
そのためには <kbd>hugo undraft</kbd> コマンドを使用します。

その前に下書きステータスとなっているページ原稿データがどれなのかを <kbd>hugo list drafts</kbd> で確認しておきましょう。

```dos
D:\workspace\hugo\yourname.github.io> hugo list drafts
post\sample01.md
page\about.md
```

前回のポストで作成したデータが下書き状態であることがわかります。
これらに対して <kbd>hugo undraft</kbd> コマンドを適用します。

```dos
D:\workspace\hugo\yourname.github.io> hugo undraft content/page/about.md
D:\workspace\hugo\yourname.github.io> hugo undraft content/post/sample01.md
```

`new` でページ原稿データを作成したときと違って `content` ディレクトリの指定も必要となっていることに注意してください。

`undraft` コマンドは、操作対象となったページ原稿データのフロントマターにおいて
`draft = false` とするとともに、`date` 行の日時情報をコマンド実行時点のものに書き換えます。
フロントマターを書き換えているだけですので、
コマンドを実行する代わりに対象ページ原稿データを直接手で書き換えても同じ効果を得られます。

もし誤って公開ステータスにしてしまった場合は、手で `draft = true` に書き戻す必要があります。

### 公開用データを生成する

いよいよ公開用データを生成します。
念のため、下書きステータスのページ原稿データが残っていないかを確認しましょう。

```dos
D:\workspace\hugo\yourname.github.io> hugo list drafts
```

漏れがなければ <kbd>hugo</kbd> を引数なしで実行します。

```dos
D:\workspace\hugo\yourname.github.io>hugo
Started building sites ...
Built site for language ja:
0 draft content
0 future content
0 expired content
2 regular pages created
5 other pages created
0 non-page files copied
3 paginator pages created
0 tags created
1 categories created
total in 187 ms
```

コマンド実行後、プロジェクト作業ディレクトリに `public` ディレクトリが増えていることを確認してください。
この `public` 配下が実際に公開されるデータの置き場（生成場所）となります。


```dos
├─layouts
├─public  ←----------------------
│  ├─2017
│  │  └─01
│  │      └─sample01
│  ├─about
│  ├─categories
│  │  └─サンプル
│  │      └─page
│  │          └─1
│  ├─css
│  │  └─highlightjs-themes
│  ├─fonts
│  ├─js
│  ├─page
│  │  ├─1
│  │  └─page
│  │      └─1
│  └─post
│      └─page
│          └─1
├─static
```


## GitHub リポジトリに連携する

前項の手順で作成した `public` ディレクトリを GitHub リポジトリ `yourname.github.io` に連携させます。

このとき `public` ディレクトリ配下のみを git で管理するという簡便な手もありますが、
ここではページ原稿データ自体も git で管理したいため、

1. プロジェクト作業ディレクトリ
1. `public` 配下

はそれぞれ別個のリポジトリに連携させるようにします。

具体的には `public` 配下は git submodule で管理します。

```dos
D:.
│
├─archetypes
├─content
├─data
├─layouts
├─public          # ここだけ git submodule で管理する
├─static
└─themes          # 実質的にはここも submodule 管理される
```

### メイン作業ディレクトリを git 管理下に置く

`public` 配下は邪魔となるため、あらかじめ全削除しておきます。
後で再度 <kbd>hugo</kbd> コマンドを実行するので問題ありません。

```dos
D:\workspace\hugo\yourname.github.io>rmdir /Q /S public
```

その後、<kbd>git init</kbd> 等の一連の追加・コミット作業を行います。

```dos
D:\workspace\hugo\yourname.github.io>git init
Initialized empty Git repository in D:/workspace/hugo/yourname.github.io/.git/

D:\workspace\hugo\yourname.github.io>git add .

D:\workspace\hugo\yourname.github.io>git status -s
A  config.toml
A  content/page/about.md
A  content/post/sample01.md
A  themes/hugo-phlat-theme

D:\workspace\hugo\yourname.github.io>git commit -m "1st commit"
[master 3957caa] 1st commit
 4 files changed, 54 insertions(+)
 create mode 100644 config.toml
 create mode 100644 content/page/about.md
 create mode 100644 content/post/sample01.md
 create mode 160000 themes/hugo-phlat-theme

D:\workspace\hugo\yourname.github.io>git status
On branch master
nothing to commit, working tree clean
```

ページ原稿データのソースファイルの公開が必要ならば、
さらに GitHub 等の外部リポジトリへの連携作業が必要となりますが、ここでは割愛します。

### public ディレクトリを git submodule 管理下に置く

<kbd>git submodule add</kbd> コマンドですでに作成済みの `yourname.github.io` を `public` フォルダとして取り込みます。
言うまでもありませんが `yourname` はあなたの GitHub アカウント名で読み替えてください。

```dos
D:\workspace\hugo\yourname.github.io>git submodule add https://github.com/yourname/yourname.github.io.git public
Cloning into 'D:/workspace/hugo/yourname.github.io/public'...
warning: You appear to have cloned an empty repository.
fatal: You are on a branch yet to be born
Unable to checkout submodule 'public'

```

warning や fatal のメッセージが出ますが、これは GitHub にあるリモートリポジトリが空であることによるものです。
ここでは無視して構いません。

## GitHub Pages に公開する

これで公開する準備が整いました。実際に公開してみましょう。

### 公開用データを生成する

これは先ほど示した手順を再度繰り返すのみです。

```dos
D:\workspace\hugo\yourname.github.io>hugo
Started building sites ...
Built site for language ja:
0 draft content
0 future content
0 expired content
2 regular pages created
5 other pages created
0 non-page files copied
3 paginator pages created
0 tags created
1 categories created
total in 187 ms
```

これで `public` ディレクトリ配下にビルドされた生成ファイルが追加されます。

### `public` 配下をコミットしてプッシュする

最後の手順として submodule となっている `public` 配下のファイルをコミットして GitHub にプッシュします。

```dos
D:\workspace\hugo\yourname.github.io>cd public
D:\workspace\hugo\yourname.github.io\public>git add -A

D:\workspace\hugo\yourname.github.io\public>git commit -m "1st build site data"
（※出力内容は省略）

D:\workspace\hugo\yourname.github.io\public>git push origin master
（※出力内容は省略）
```

これで https://youname.github.io/ にてページが公開されているはずです。
実際にアクセスしてみて確認してみてください。
