+++
draft = false
description = "HUGOで静的ページを作成して GitHub pages で公開するまでの手順"
author = "holywise"
isCJKLanguage = true
tags = [
  "Hugo", "Windows", "GitHub"
]
date = "2016-12-29T01:55:40+09:00"
slug = "how_to_setup_hugo_site"
categories = [
  "環境構築",
]
title = "HUGOで静的ページサイトを構築する (1/3)"

+++

## そもそも HUGO って何？

とてもざっくり言うと [HUGO](https://gohugo.io) とは Markdown 等の形式で書いたドキュメント群を、
`hugo` コマンド一発で静的なウェブサイトとしてビルドするツールです。
ビルドされた一連のファイル群は Apache などが稼働するウェブサーバーにそのまま配置することが可能です。
バージョン 0.17 からは多言語サイトの構築にも対応しました。
そのためマニュアル等のドキュメントを複数の言語で提供したい、などといったことも可能です。
もちろん静的ページ（要はHTMLファイル）を生成するものであるため、サーバー側にPHPやRuby等の実行環境は必要ありません。

HUGOは、主に次のような目的のサイト構築に適していると言えるでしょう。

- 企業サイトのランディングページ
- OSSプロジェクトのドキュメントやマニュアル
- フリーランサーのポートフォリオページ
- 個人ブログ

`hugo` コマンド自体は [Go](https://golang.org/) 言語で作成されており、
Windows / Mac OS / Linux 等さまざまな環境上で動作します。
ビルド速度も高速です。

ちなみに今ご覧のこのサイト自体も HUGO を使って構築されています。
今回以降の一連のポストでは、Windows 上にこれらのページを作成する環境を構築し、
最終的に [GitHub Pages](https://pages.github.com/) で公開するまでの手順を紹介していきます。

<!--more-->

## 構築を行う環境および前提条件

筆者の開発環境である Windows10 Pro 64bit (Build 1511) に環境を構築していった例で示していきます。
Windwos の特定のバージョンに依存するような処理は行わないので、Windows7 等でも特に問題はないはずです。

以下のツールがすでにインストールされていることを前提とします。

- [git for windows](https://git-scm.com/downloads)

各種コマンド実行は cmd.exe によるコマンドプロンプトから行います。cygwin や bash は今回使用しません。
git は CLI である必要は無く、 [SourceTree](https://ja.atlassian.com/software/sourcetree) などのような
GUI アプリの方が使い慣れているというのであれば、そちらを利用しても構いません。

また [GitHub](https://github.com/) アカウントはすでに持っているものとします。
これは GitHub Pages でページを公開するためには必須となります。
もしまだアカウントを取得していないのであれば [Sign up](https://github.com/join?source=header-home) してください。

## 構築作業の全景

この一連のポストでは以下の手順で進めていくことになります。

1. [ ] **HUGOをインストールする**
1. [ ] **プロジェクト用の作業ディレクトリを作る**
1. [ ] **作業ディレクトリに各種の初期設定を行う**
1. [ ] [作業ディレクトリにページデータを作成する]({{< relref "howto_add_hugo_pages.md" >}})
1. [ ] [GitHub Pages に連携する]({{< relref "howto_push_hugo_site.md" >}})

当ポストは3番目までを示します。
4番以降の手順はリンク先のポストをご覧ください。

## HUGO をインストールする

HUGO のインストールは実に簡単で、実行ファイルを path の通ったところに置けばいいという、ただそれだけです。

### バイナリのダウンロード

<https://github.com/spf13/hugo/releases>

上記リンク先が最新版リリース版のダウンロードサイトです。

2016年12月20日現在、最新版は 0.18 です。Windows 用のバイナリは

- `hugo_0.18_Windows-32bit.zip`
- `hugo_0.18_Windows-64bit.zip`

のいずれかとなりますので、インストール先の環境に応じて適した方をダウンロードします。
以下の説明では 64bit 版をダウンロードしたものとして記述していきます。

### バイナリのインストール

上記リンク先からダウンロードした zip ファイルを展開します。次の三つのファイルが入っているはずです。

- `hugo_0.18_windows_amd64.exe`
- `LICENSE.md`
- `README.md`

`hugo_0.18_windows_amd64.exe` を `hugo.exe` にリネームして path の通った任意のディレクトリにコピーします。

もちろん `hugo.cmd` という名前のバッチファイルを作り、その中で `hugo_0.18_windows_amd64.exe` を呼び出すようにしても良いですし、
<kbd>mklink</kbd> を利用して `hugo_0.18_windows_amd64.exe` に対して `hugo.exe` という名前でシンボリックリンクもしくはハードリンクを作るという方法もあります。
そのあたりはお好みで。


### バイナリの動作確認

コマンドプロンプトを開きます。
<kbd>hugo version</kbd> を実行して、バージョン番号が表示されれば正常にインストールされています。

```dos
D:\temp\hugotest>hugo version
Hugo Static Site Generator v0.18 BuildDate: 2016-12-24T12:10:44+09:00
```

## プロジェクト用の作業ディレクトリを作る

静的サイトを構築するためのプロジェクト作業ディレクトリを任意の場所に作成します。
今回の例では `D:\workspace\hugo\` の下に `yourname.github.io` を作業ディレクトリとして作成することにします。
`yourname` の部分はあなたの GitHub アカウント名に合わせておいてください。
これ以降 `yourname` の表記が出てきた場合は、適宜ご自身のアカウント名に読み替えてください。

まず、親ディレクトリを作ってそこに移動します。

```dos
D:\workspace> mkdir hugo
D:\workspace> cd hugo
D:\workspace\hugo>
```

HUGO でプロジェクト作業ディレクトリを作るためには <kbd>hugo new site</kbd> コマンドを実行します。
これにより作業ディレクトリが新規作成されます。

```dos
D:\workspace\hugo> hugo new site yourname.github.io
Congratulations! Your new Hugo site is created in "D:\\workspace\\hugo\\yourname.github.io".

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/, or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.
```

これでプロジェクト作業ディレクトリができました。
作業ディレクトリは以下のような構造になっています。

```dos
D:\workspace\hugo\yourname.github.io> tree /F

D:.
│  config.toml
│
├─archetypes
├─content
├─data
├─layouts
├─static
└─themes
```

６つの空ディレクトリ `archetypes` `content` `data` `layouts` `static` `themes` と、
作業ディレクトリのトップに `config.toml` が作成されます。これは設定ファイルです。
各ディレクトリの意味や目的は次回のポストで説明しますので、ここではそのまま先に進んでください。

### config.toml の初期状態

```toml
languageCode = "en-us"
baseurl = "http://replace-this-with-your-hugo-site.com/"
title = "My New Hugo Site"
```

`config.toml` の初期状態は上記のようなわずか3行のファイルです。

## 作業ディレクトリに各種の初期設定を行う

### テーマをインストールする

HUGO はテーマが存在しないとサイトデータを生成することができませんが、HUGO にはデフォルトテーマは付属していません。
何はともあれ、まずはテーマを選んでインストールすることが設定の最初に行うべきことになります。

テーマは
[Hugo Themes Site](http://themes.gohugo.io/)
でいろいろ公開されています。

#### テーマを選択する基準

今回は当サイトのような個人ブログを目的とします。
そういう場合は
[タグとして"blog"が設定されているもの](http://themes.gohugo.io/tags/blog)
の中から選ぶと良いでしょう。
その上でサイトの目的に応じたものかつお好みのデザインのテーマを選びます。

なおテーマによっては若干のクセがあって `/content` 配下が特定のディレクトリ構造でなければならない、などといった制限がつくものがあります。
`/content` 配下の構造が、サイトを公開する際に各ページのURLへ影響しますので、そのあたりも勘案した上でテーマを選択してください。

以降では当サイトでも使用している
[Hugo Theme: Phlat Theme](http://themes.gohugo.io/hugo-phlat-theme/)
を採用したものとして、例示していきます。

#### テーマのインストール

テーマごとにインストールの方法が記述されています。
大抵は `/themes` ディレクトリの下で <kbd>git clone</kbd> するだけです。
今回の Phlat テーマもその方法となります。

```dos
D:\workspace\hugo\yourname.github.io> cd themes
D:\workspace\hugo\yourname.github.io\themes> git clone https://github.com/nraboy/hugo-phlat-theme
Cloning into 'hugo-phlat-theme'...
remote: Counting objects: 433, done.
remote: Total 433 (delta 0), reused 0 (delta 0), pack-reused 433
Receiving objects: 100% (433/433), 1.59 MiB | 39.00 KiB/s, done.
Resolving deltas: 100% (232/232), done.
Checking connectivity... done.
```

### 設定ファイル config.toml を編集する

`/config.toml` が設定ファイルとなります。
以下に設定例を示します。

```toml
#
# config.toml
#

DefaultContentLanguage = "ja"             # ビルド時のデフォルト言語指定
languageCode = "ja-JP"                    # 公開するページの言語指定

baseurl = "https://yourname.github.io/"   # 公開時のベースとなるURL
title = "My New Hugo Site"                # サイトのメインタイトル
theme = "hugo-phlat-theme"                # サイトで使用するテーマ名

[permalinks]
  post = "/:year/:month/:slug/"           # post タイプページのパーマネントリンク構成
  page = "/:slug/"                        # page タイプページのパーマネントリンク構成

[taxonomies]
  tag = "tags"                            # タググループ名
  category = "categories"                 # カテゴリグループ名

[params]
  keywords = ["programming", "developer"] # 生成ページに keywords 指定がないときのデフォルト値 (meta keywords)
  description = "my tech notes"           # 生成ページに description 指定がないときのデフォルト値 (meta description)

[[menu.header]]
  name = "Home"
  weight = 1
  url = "/"

[[menu.header]]
  name = "About"
  weight = 2
  url = "/about/"
```

使用するテーマによっては必ず記述しなければいけない項目が指定されていることがあります。
今回の Phlat テーマでは、最低限、上記のような項目を指定しておかなくてはいけません。

[HUGOで静的ページサイトを構築する (2/3)]({{< relref "howto_add_hugo_pages.md" >}}) に続きます。
