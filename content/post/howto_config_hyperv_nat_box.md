+++
title = "Windows10 Client Hyper-V にて CentOS7 box に NAT 環境を構築する (2/2)"
categories = ["環境構築"]
tags = ["Windows","Hyper-V","CentOS"]
isCJKLanguage = true
author = "holywise"
date = "2017-01-18T22:58:50+09:00"
slug = "howto_config_hyperv_nat_box"
draft = false
description = "Windows10 Client Hyper-V にて CentOS7 box に NAT 環境を構築するまでの手順"

+++

## 構築作業の全景

この一連のポストでは以下の手順で進めていくことになります。

1. [x] [ホスト側に仮想ネットワークアダプタを作成する]({{< relref "howto_setup_hyperv_box.md" >}})
2. [ ] **ゲスト側にネットワーク設定を行う** (←)

[前回のポスト]({{< relref "howto_setup_hyperv_box.md" >}})では前半の手順を終えました。
当ポストでは後半の手順を示します。

{{< figure src="/image/HyperV_NAT_and_HostOnly.png" alt="Network構成図" caption="ネットワーク構成図" >}}

<!--more-->

## ゲスト用の Hyper-V box を作成する

Hyper-V マネージャー を起動し、新規仮想マシンを作成します。
その際に、仮想マシンにはネットワークアダプタを２つ作成し、
一方には仮想スイッチに `NAPT` が、もう一方には仮想スイッチに `HostOnly` が接続されるようにしておきます。

{{< figure src="/image/HyperV_create_wizard.png" alt="Hyper-V 仮想マシンの新規作成ウィザード" caption="新規作成ウィザードのネットワーク構成画面" >}}


{{< figure src="/image/HyperV_machine_property.png" alt="Hyper-V 仮想マシンのプロパティ" caption="仮想マシンのプロパティ画面" >}}


## ゲストOSをインストールし、ネットワーク設定を行う

CentOS7 のインストール自体は、インストーラーウィザードが指定する手順どおりに行います。
その際に「ネットワークとホスト名」の設定で `NAPT` および `HostOnly` の仮想スイッチに繋がっている
２つの「イーサネット」について、それぞれ次のように設定します。

### NAPT

#### IPv4のセッティング

<table class="table table-striped">
  <tr>
    <th style="width: 8em;">項目</th>
    <th>値</th>
    <th>備考</th>
  </tr>
  <tr>
    <td>方式</td>
    <td>手動</td>
    <td></td>
  </tr>
  <tr>
    <td>アドレス</td>
    <td>10.0.1.100</td>
    <td>NAPT用仮想スイッチに <kbd>New-NetNat</kbd> で割り当てた 10.0.1.0/24 に属する任意のもの</td>
  </tr>
  <tr>
    <td>ネットマスク</td>
    <td>255.255.255.0</td>
    <td></td>
  </tr>
  <tr>
    <td>ゲートウェイ</td>
    <td>10.0.1.1</td>
    <td>NAPT用仮想スイッチに <kbd>New-NetIPAddress</kbd> で割り当てたIPアドレスを指定</td>
  </tr>
  <tr>
    <td>DNSサーバー</td>
    <td>192.168.1.1</td>
    <td>ホストPCが物理NICから参照しているDNSサーバーを指定</td>
  </tr>
</table>

### HostOnly

#### IPv4のセッティング

<table class="table table-striped">
  <tr>
    <th style="width: 8em;">項目</th>
    <th>値</th>
    <th>備考</th>
  </tr>
  <tr>
    <td>方式</td>
    <td>手動</td>
    <td></td>
  </tr>
  <tr>
    <td>アドレス</td>
    <td>192.168.10.100</td>
    <td>ホストオンリー用仮想スイッチに <kbd>New-NetIPAddress</kbd> で割り当てた 192.168.10.1/24 と同じ空間に属する任意のもの</td>
  </tr>
  <tr>
    <td>ネットマスク</td>
    <td>255.255.255.0</td>
    <td></td>
  </tr>
  <tr>
    <td>ゲートウェイ</td>
    <td></td>
    <td>空欄のまま</td>
  </tr>
  <tr>
    <td>DNSサーバー</td>
    <td></td>
    <td>空欄のまま</td>
  </tr>
</table>

### 設定の確認

インストール終了後、ゲストOSにログインしてネットワーク設定の確認を行います。
CentOS7 の場合は <kbd>ip route</kbd> を使用します。

```sh
$ ip route
default via 10.0.1.1 dev eth0  proto static  metric 100
10.0.1.0/24 dev eth0  proto kernel  scope link  src 10.0.1.100  metric 100
192.168.10.0/24 dev eth1  proto kernel  scope link  src 192.168.10.100  metric 100
```

上記のように `default` ルートが `10.0.1.1` (「NAPT IPv4のセッティング」で指定したゲートウェイ) に向いていることを確認してください。
その後 <kbd>ping -c3 www.google.com</kbd> などとして、外部ネットワークへの到達性をチェックすると良いでしょう。

なお標準状態では仮想マシンからホストPCへの ping はWindowsファイアーウォールにブロックされてしまいます。
そのため <kbd>ping 192.168.10.1</kbd> などとしても 100% paket loss となりますが、それで正常です。

仮想マシンからホストPCへの ping の到達性を確認したい場合は、
「セキュリティが強化されたWindowsファイアウォール」
（<kbd><kbd>Win</kbd> + <kbd>R</kbd></kbd>で開くダイアログから<kbd>wf.msc</kbd>で起動）
から「受信の規則」→「仮想マシンの監視（エコー要求 - ICMPv4受信）」の「規則を有効化」してください。

{{< figure src="/image/windows_firewall_icmp.png" alt="セキュリティが強化されたWindowsファイアウォール" caption="仮想マシンの監視ルールの有効化" >}}

なおこのルールはプライベート・パブリック・ドメインのすべてのネットワークプロファイルに対して icmp 受信を開放するものなので、
到達性の確認ができた時点で再度無効化しておくことをお勧めします。

これでゲスト側のネットワーク設定は完了です。

## 【任意】開発環境用にWindowsファイアウォールの設定を変更する

前項までの手順を進めることで、前回のポストで挙げた

- ゲストの CentOS 環境へはホストとなる Windows10 からしかアクセスできないようにしたい
- ゲストの CentOS 環境の中からは自由に外部（インターネット）へアクセスしたい

という目的自体はすでに達成しています。

ところでこの仮想マシンをウェブアプリ等の開発環境としたい場合、
仮想マシンからホストPCへの通信が可能となっていて欲しいことがよくあります。
例えば仮想マシン上で動作するPHPアプリケーションを [Xdebug](https://xdebug.org/) でリモートデバッグしたい、
などというケースです。

ところが前回・今回の一連の手順で示したネットワーク構成では、
「HostOnly」「NAPT」ともに「パブリックネットワーク」として扱われるようになるため、
標準状態では仮想マシンからホストPCへの接続が厳しく制限されてしまいます。

ここではホストPCが、仮想マシン上で動作している Xdebug からの接続を受け付けるように、
Windowsファイアウォールを設定する例を示します。

### Xdebug から接続されるポート番号を確認する

仮想マシンの `/etc/php.d` 配下に、次のような内容の `xdebug.ini` が置かれているケースで考えます。

```ini
; Enable xdebug extension module
zend_extension=/usr/lib64/php/modules/xdebug.so

; see http://xdebug.org/docs/all_settings
xdebug.remote_enable=1
xdebug.remote_connect_back=1
xdebug.remote_port=9000
```

この設定では、仮想マシンからホストPCの9000番ポートへのコールバックが発生することになります。
つまり仮想マシンからホストPCの9000番ポートが通過できるように設定すれば良いということになります。

### Windowsファイアウォールを設定する

<kbd><kbd>Win</kbd> + <kbd>R</kbd></kbd>で開くダイアログから<kbd>wf.msc</kbd>で
「セキュリティが強化されたWindowsファイアウォール」を起動します。

{{< figure src="/image/run_wf_msc.png" alt="コマンドを指定して実行" caption="コマンドを指定して実行" class="origin" >}}

受信の規則から「新しい規則」を選択して「新規の受信の規則ウィザード」に沿って各種設定を行っていきます。

#### 規則の種類

作成するファイアウォールの規則の種類を選択します。

今回は仮想マシン上のプログラムのリモートデバッグを目的としているため、

- `カスタム`

を選択して、細かく指定をしていきます。

{{< figure src="/image/wf_rule_wizard_01.png" alt="受信の規則ウィザード" caption="規則の種類" >}}


#### プログラム

待ち受けるプログラムを特定しても構いませんが、
IDEなどはバージョンのアップグレードなどでプログラムの配置位置が変わったりすることと、
後工程のスコープ指定で対象とするネットワークを限定できるため、

- `すべてのプログラム`

を選択します。

{{< figure src="/image/wf_rule_wizard_02.png" alt="受信の規則ウィザード" caption="プログラム" >}}


#### プロトコルとポート

先ほど確認したとおり、Xdebugの待ち受けは9000番ポートとなります。

- プロトコルの種類：`TCP`
- ローカルポート：`9000`
- リモートポート：`すべてのポート`

を選択します。

{{< figure src="/image/wf_rule_wizard_03.png" alt="受信の規則ウィザード" caption="プロトコルとポート" >}}

#### スコープ

前項までの行程で設定を行った `HostOnly` ネットワークからのみ、接続を受け付けるようにします。

- ローカルIPアドレス：`192.168.10.1`
- リモートIPアドレス：`192.168.10.0/24`

を指定します。

{{< figure src="/image/wf_rule_wizard_04.png" alt="受信の規則ウィザード" caption="スコープ" >}}

#### 操作

パケットを通したいので

- `接続を許可する`

を選択します。

{{< figure src="/image/wf_rule_wizard_05.png" alt="受信の規則ウィザード" caption="操作" >}}

#### プロファイル

これまでの手順で設定した `HostOnly` ネットワークアダプタは「パブリックネットワーク」として扱われていますので、

- `パブリック`

を選択します。

{{< figure src="/image/wf_rule_wizard_06.png" alt="受信の規則ウィザード" caption="プロファイル" >}}

#### 名前

最後に規則に任意の名前を付けます。
ここの例では「Xdebug」としています。

最後に「完了」をクリックしてください。

{{< figure src="/image/wf_rule_wizard_07.png" alt="受信の規則ウィザード" caption="名前" >}}

以上で、すべてのネットワーク設定が完了しました。
