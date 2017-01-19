+++
date = "2016-12-29T01:55:48+09:00"
slug = "how_to_add_pages_hugo_site"
description = "Windows 環境で HUGO で静的ページを作成して GitHub Pages でブログとして公開するまでの手順について"
draft = false
categories = ["環境構築"]
tags = ["Hugo", "Windows", "GitHub"]
isCJKLanguage = true
author = "holywise"
title = "HUGOで静的ページサイトを構築する (2/3)"
+++

## 構築作業の全景

この一連のポストでは以下の手順で進めていくことになります。

1. [x] HUGOをインストールする
1. [x] プロジェクト用の作業ディレクトリを作る
1. [x] 作業ディレクトリに各種の初期設定を行う
1. [ ] **作業ディレクトリにページデータを作成する** (←)
1. [ ] [GitHub Pages に連携する]({{< relref "howto_push_hugo_site.md" >}})

[前回のポスト]({{< relref "howto_setup_hugo_site.md" >}})で3番目の手順まで終えました。
当ポストでは4番目の手順を示します。

<!--more-->

## 作業ディレクトリにページデータを作成する

前回のポストまでの手順で下記のような作業ディレクトリ構造ができあがっています。

```dos
D:.
│  config.toml    # サイト設定ファイル
│
├─archetypes      # 新規ページ原稿のひな形テンプレートを置くところ
├─content         # ページ原稿を置くところ
├─data            # Hugo制御用の設定ファイルを置くところ
├─layouts         # 生成されるサイトを構成する各htmlテンプレート
├─static          # cssやjavascript等の静的素材を置くところ
└─themes          # テーマファイル群を置くところ
```

`/layouts` や `/static` は主にインストールしたテーマをカスタマイズしたいときに使用します。
`/content` の下がユーザが markdown 形式等で作成していくページ原稿の置き場となります。

ページ原稿データはコマンドラインから <kbd>hugo new</kbd> コマンドを使用して作成しますが、
このとき作成されるページ原稿のひな形として `/archetypes` 配下に置いたファイルが使用されます。
ただし今回導入したテーマ
[Hugo Theme: Phlat Theme](http://themes.gohugo.io/hugo-phlat-theme/)
は、テーマの中に `archetypes/default.md` を持っているため、
ユーザが自力で `/archetypes` の下にファイルを作らなくてもそちらが参照されます。

### 原稿データを新規に追加する

<kbd>hugo new</kbd>コマンド実行は作業ディレクトリのルートで行ってください。
ここでは引数の例として `post/sample01.md` と `page/about.md` を指定します。

```dos
D:\workspace\hugo\youname.github.io> hugo new post/sample01.md
D:\workspace\hugo\youname.github.io\content\post\sample01.md created
D:\workspace\hugo\youname.github.io> hugo new page/about.md
D:\workspace\hugo\youname.github.io\content\page\about.md created
```

`/contens` の下に新たに `post` ディレクトリが作られ、その下に `sample01.md` が作成されたことがわかります。
同様に `page` ディレクトリとその下に `about.md` が作成されています。

```dos
D:.
│  config.toml
│
├─archetypes
├─content
│  ├─page                # pageセクション
│  │      about.md
│  │
│  └─post                # postセクション
│         sample01.md
│
├─data
```

`/content` 直下のディレクトリは「セクション」と呼ばれ、特別な意味があります。
どのようなセクション名が使えるかは使用するテーマに依存します。
今回採用した Phlat テーマでは `post` および `page` の２つのセクションを持つことができるようになっています。

Phlat テーマの場合は `post` セクションの下がブログのいわゆる「エントリー」に該当し、
`page` セクションの下は「このサイトについて」などの固定ページに該当します。

セクション名ディレクトリの下の原稿データファイル名は自由に付けられます。
何も設定しなければ基本的にここでの原稿データファイル名（拡張子を除く）が公開するサイトのURLに使われます。

### 原稿データを編集する

`page/about.md` は次のような内容になっているはずです。

```markdown
+++
date = "2016-12-26T21:10:55+09:00"
title = "about"
draft = true

+++
```

`+++` で挟まれた行はフロントマターと言って、そのページの作成日付等のメタ情報をTOML形式で記述します。
`date` の内容は自動生成されており <kbd>hugo new</kbd> を実行した日時になっています。
`draft = true` はこの原稿データが下書きステータスであることを表しています。

ページ本文は2回目に出現する `+++` 以降に記述していきます。
以下にサンプルを示します。

```markdown
+++
date = "2016-12-26T21:10:55+09:00"
title = "about"
draft = true
isCJKLanguage = true

+++

## このサイトについて

このサイトはサンプルです。
```

同様に `post/sample01.md` の方も編集します。

```markdown
+++
date = "2016-12-26T21:09:51+09:00"
title = "sample01"
draft = true
isCJKLanguage = true
categories = ["サンプル"]

+++

## サンプルポスト

今日はこんなことがありました。
```

なおフロントマターに `isCJKLanguage = true` を追加しています。
これは本文が日本語（or 中国語 or 韓国語）であることを明示するものです。
必須ではありませんが、日本語で原稿を書くなら付けておくと良いでしょう。

また Hugo は markdown の処理に [Blackfriday](https://github.com/russross/blackfriday) を使用しています。
標準的な markdown の記法に若干の拡張がされているので、どのような記法が使えるかを上記のリンク先で見ておくことをお勧めします。

### ローカル環境でプレビューを見る

Hugo はそれ自身でウェブサーバーとして振る舞うことができます。
作業ディレクトリのルートで <kbd>hugo server</kbd> コマンドを実行することで、
ローカルホストにブラウザでアクセスしてプレビューを見ることができます。

```dos
D:\workspace\hugo\youname.github.io> hugo server -D
Started building sites ...
Built site for language ja:
2 of 2 drafts rendered
0 future content
0 expired content
2 regular pages created
5 other pages created
0 non-page files copied
3 paginator pages created
1 categories created
0 tags created
total in 32 ms
Watching for changes in D:\workspace\hugo\yourname.github.io\{data,content,layouts,static,themes}
Serving pages from memory
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop

```

`-D` オプションを付けたのはフロントマターで `draft = true` となっているページも出力対象とするためです。

上記のように特にエラーがなく `Web Server is available` と表示されていれば

http://localhost:1313/

にブラウザでアクセスしてみてください。
正常に動いていれば


{{< figure src="/image/hugo_test_sample.png" alt="HUGO Sample" caption="Hugoサンプル表示" >}}


のように表示されるはずです。

また　Hugo は <kbd>hugo server</kbd> で実行したとき、ライブリロード機能が有効になります。
ライブリロード機能とは、サーバー機能が実行中にページデータ等を編集すると即座に表示中のページがリロードされて、
編集内容が自動的にプレビューに反映されるものです。Hugo サーバーを実行したまま `post/sample01.md` を編集してみてください。

Hugo サーバーを終了するには <kbd><kbd>Ctrl</kbd>+<kbd>C</kbd></kbd> を押してください。

[HUGOで静的ページサイトを構築する (3/3)]({{< relref "howto_push_hugo_site.md" >}}) に続きます。
