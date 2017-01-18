+++
date = "2017-01-18T22:58:41+09:00"
title = "Windows10 Client Hyper-V にて CentOS7 box に NAT 環境を構築する (1/2)"
description = "Windows10 Client Hyper-V にて CentOS7 box に NAT 環境を構築するまでの手順"
tags = ["Windows","Hyper-V","CentOS"]
isCJKLanguage = true
author = "holywise"
slug = "howto_setup_hyperv_nat_box"
draft = false
categories = ["環境構築"]

+++

## 目的とゴール

Windows10 Pro では Client Hyper-V と呼ばれる仮想マシン環境を構築することができます。

この一連のポストは、Windows10 Pro をホストとした Hyper-V による CentOS7 の仮想マシンを、
ウェブアプリ等のローカル開発環境とするためのネットワーク環境を整える手順をステップバイステップで示すことを目的とします。

このポストで紹介する手順に沿ってネットワーク環境を構築することで

- ゲストの CentOS 環境へはホストとなる Windows10 からしかアクセスできないようにしたい
- ゲストの CentOS 環境の中からは自由に外部（インターネット）へアクセスしたい（yum install 等のため）

という要求を満たすことができるようになります。

## 構築を行う環境および前提条件

- ホストOS : Windows10 Pro バージョン 1511 ビルド 10586 以降
- ホストOSで Hyper-V が有効化済みであること
- ゲストOS : CentOS 7

各種設定の操作は主に PowerShell を利用します。

<!--more-->

## 実装の方針

1. ホストPCの中に仮想ネットワークアダプタを２つ作り、１つはホストオンリー接続用とし、もう１つはNAPT接続用とする。
2. ゲスト側にもホストオンリーアダプタへの接続用とNATアダプタ接続用の2つのネットワークインターフェースを設定する。
3. NAPT接続用のアダプタにはNATオブジェクトの割り当てを行い、NAT用ネットワークとして振る舞えるようにする。
4. NAPT接続用アダプタはホストPCの物理NICを経由して、インターネットへの接続を可能となる。

{{< figure src="/image/HyperV_NAT_and_HostOnly.png" alt="Network構成図" caption="ネットワーク構成図" >}}


以降の手順で、上図のようなネットワーク環境を構築していきます。

## 構築作業の全景

この一連のポストでは以下の手順で進めていくことになります。

1. [ ] **ホスト側に仮想ネットワークアダプタを作成する** (←)
2. [ ] [ゲスト側にネットワーク設定を行う]({{< relref "howto_config_hyperv_nat_box.md" >}})

当ポストでは前半の手順を示します。

## 構築の手順

### ホスト側でホストオンリー用仮想ネットワークアダプタを作成する

ネットワークアダプタは Hyper-V マネージャーから GUI で操作していく方法と、
PowerShell から CUI で操作する方法の、いずれかで作成できます。

ここでは CUI から作成する方法で示していきます。

まず__管理者権限__で PowerShell コンソールを開きます。
次の <kbd>New-VMswitch</kbd> コマンドを実行して、内部用仮想ネットワークアダプタを作成します。
これは Hyper-V マネージャで言えば、新規の内部仮想スイッチを作成したことに該当します。

```powershell
PS C:\WINDOWS\system32> New-VMswitch -SwitchName "HostOnly" -SwitchType Internal
```

<kbd>Get-NetAdapter</kbd> コマンドでネットワークアダプタ一覧を表示させます。
ここで今作成した `HostOnly` の ifIndex の値をメモしておいてください。
次の例では「20」となっています。

```powershell
PS C:\WINDOWS\system32> Get-NetAdapter

Name                      InterfaceDescription                    ifIndex Status       MacAddress             LinkSpeed
----                      --------------------                    ------- ------       ----------             ---------
Wi-Fi                     Qualcomm Atheros AR9485 Wireless Net...      19 Disconnected 48-**-**-**-**-F4          0 bps
vEthernet (HostOnly)      Hyper-V Virtual Ethernet Adapter             20 Up           00-**-**-**-**-00        10 Gbps
イーサネット               Realtek PCIe GBE Family Controller           14 Up           D8-**-**-**-**-CF       100 Mbps
```

<kbd>New-NetIPAddress</kbd> コマンドで、この `HostOnly` ネットワークアダプタに対する IP アドレスを設定します。
ここの例では `192.168.10.1/24` を割り当てています。
また `-InterfaceIndex` オプションで先ほどメモした ifIndex の値を指定しています。

```powershell
PS C:\WINDOWS\system32> New-NetIPAddress -IPAddress 192.168.10.1 -PrefixLength 24 -InterfaceIndex 20

IPAddress         : 192.168.10.1
InterfaceIndex    : 20
InterfaceAlias    : vEthernet (HostOnly)
AddressFamily     : IPv4
Type              : Unicast
PrefixLength      : 24
PrefixOrigin      : Manual
SuffixOrigin      : Manual
AddressState      : Tentative
ValidLifetime     : Infinite ([TimeSpan]::MaxValue)
PreferredLifetime : Infinite ([TimeSpan]::MaxValue)
SkipAsSource      : False
PolicyStore       : ActiveStore

IPAddress         : 192.168.10.1
InterfaceIndex    : 20
InterfaceAlias    : vEthernet (HostOnly)
AddressFamily     : IPv4
Type              : Unicast
PrefixLength      : 24
PrefixOrigin      : Manual
SuffixOrigin      : Manual
AddressState      : Invalid
ValidLifetime     : Infinite ([TimeSpan]::MaxValue)
PreferredLifetime : Infinite ([TimeSpan]::MaxValue)
SkipAsSource      : False
PolicyStore       : PersistentStore
```

デフォルトゲートウェイやDNSの設定は特に必要ありません。

### ホスト側でNAPT用仮想ネットワークアダプタを作成する

こちらはNAPT用であるため、前項で示した手順に加えて物理NICへのNAT接続設定が必要となります。

#### 内部用仮想ネットワークアダプタを作成する

まず前項と同様の手順でNAPT用の仮想ネットワークアダプタを作成します。
`HostOnly` と同様に内部用の仮想ネットワークアダプタを作成することになります。

```powershell
PS C:\WINDOWS\system32> New-VMswitch -SwitchName "NAPT" -SwitchType Internal
PS C:\WINDOWS\system32> Get-NetAdapter

Name                      InterfaceDescription                    ifIndex Status       MacAddress             LinkSpeed
----                      --------------------                    ------- ------       ----------             ---------
Wi-Fi                     Qualcomm Atheros AR9485 Wireless Net...      19 Disconnected 48-**-**-**-**-F4          0 bps
vEthernet (NAPT)          Hyper-V Virtual Ethernet Adapter #2          12 Up           00-**-**-**-**-01        10 Gbps
vEthernet (HostOnly)      Hyper-V Virtual Ethernet Adapter             20 Up           00-**-**-**-**-00        10 Gbps
イーサネット               Realtek PCIe GBE Family Controller           14 Up           D8-**-**-**-**-CF       100 Mbps
```

やはり同様にこのネットワークアダプタにIPアドレスを割り当てます。
ここの例では `10.0.1.1/24` を割り当てています。

```powershell
PS C:\WINDOWS\system32> New-NetIPAddress -IPAddress 10.0.1.1 -PrefixLength 24 -InterfaceIndex 12

IPAddress         : 10.0.1.1
InterfaceIndex    : 12
InterfaceAlias    : vEthernet (NAPT)
AddressFamily     : IPv4
Type              : Unicast
PrefixLength      : 24
PrefixOrigin      : Manual
SuffixOrigin      : Manual
AddressState      : Tentative
ValidLifetime     : Infinite ([TimeSpan]::MaxValue)
PreferredLifetime : Infinite ([TimeSpan]::MaxValue)
SkipAsSource      : False
PolicyStore       : ActiveStore

IPAddress         : 10.0.1.1
InterfaceIndex    : 12
InterfaceAlias    : vEthernet (NAPT)
AddressFamily     : IPv4
Type              : Unicast
PrefixLength      : 24
PrefixOrigin      : Manual
SuffixOrigin      : Manual
AddressState      : Invalid
ValidLifetime     : Infinite ([TimeSpan]::MaxValue)
PreferredLifetime : Infinite ([TimeSpan]::MaxValue)
SkipAsSource      : False
PolicyStore       : PersistentStore
```

#### NAPT用仮想ネットワークアダプタにNATオブジェクトを割り当てる

[New-NetNat](https://technet.microsoft.com/en-us/library/dn283361.aspx) コマンドによって
内部ネットワークアダプタに対して NAPT 機能を割り当てることができます。

```powershell
PS C:\WINDOWS\system32> New-NetNat -Name "MyNAPT" -InternalIPInterfaceAddressPrefix 10.0.1.0/24

Name                             : MyNAPT
ExternalIPInterfaceAddressPrefix :
InternalIPInterfaceAddressPrefix : 10.0.1.0/24
IcmpQueryTimeout                 : 30
TcpEstablishedConnectionTimeout  : 1800
TcpTransientConnectionTimeout    : 120
TcpFilteringBehavior             : AddressDependentFiltering
UdpFilteringBehavior             : AddressDependentFiltering
UdpIdleSessionTimeout            : 120
UdpInboundRefresh                : False
Store                            : Local
Active                           : True
```

なお `-InternalIPInterfaceAddressPrefix` オプションにはネットワークアドレス（10.0.1.**0**）が指定されていることに注意してください。

この設定により、`10.0.1.0/24` に属する内部ネットワークから NAPT によるネットワークアドレス・ポート変換が働くようになり、
この `10.0.1.0/24` ネットワーク空間の中からは、
先ほどこの仮想ネットワークアダプタに設定した `10.0.1.1` がゲートウェイとしての働きを持つようになります。
またこのゲートウェイアドレス `10.0.1.1` とホストPCの物理NICとの接続設定は明示的に行う必要はなく、自動的に接続されるようになります。

参考
: [Set up a NAT network](https://docs.microsoft.com/ja-jp/virtualization/hyper-v-on-windows/user-guide/setup-nat-network) (docs.microsoft.com)


### ここまでの確認

<kbd>Get-NetConnectionProfile</kbd> コマンドで、各ネットワークアダプタのプロファイルを知ることができます。

```powershell
PS C:\Windows\system32> Get-NetConnectionProfile

Name             : *****************
InterfaceAlias   : イーサネット
InterfaceIndex   : 14
NetworkCategory  : Private
IPv4Connectivity : Internet
IPv6Connectivity : LocalNetwork

Name             : 識別されていないネットワーク
InterfaceAlias   : vEthernet (HostOnly)
InterfaceIndex   : 20
NetworkCategory  : Public
IPv4Connectivity : NoTraffic
IPv6Connectivity : NoTraffic

Name             : 識別されていないネットワーク
InterfaceAlias   : vEthernet (NAPT)
InterfaceIndex   : 12
NetworkCategory  : Public
IPv4Connectivity : NoTraffic
IPv6Connectivity : NoTraffic
```

`HostOnly` および `NAPT` のいずれも「パブリックネットワーク」と扱われており、
「識別されていないネットワーク」になっていますが、
これはどちらのネットワークアダプタにもデフォルトゲートウェイ等の設定を行っていないためであり、
これで正常な状態です。

気になるようであれば <kbd>Set-NetConnectionProfile</kbd> コマンドでこれらをプライベートネットワークにすることもできます。

```powershell
PS C:\WINDOWS\system32> Set-NetConnectionProfile -Name "識別されていないネットワーク" -NetworkCategory Private
```

ただし、この設定はホストPCを再起動するまでしか有効ではありません。
ホストPCを再起動すると、またパブリックネットワーク状態に戻ってしまいます。

また `HostOnly` と `NAPT` のどちらかだけをプライベートにする、ということもできません。
たとえ

```powershell
PS C:\WINDOWS\system32> Set-NetConnectionProfile -InterfaceIndex 12 -NetworkCategory Private
```

のように `InterfaceIndex` を特定して設定したとしても、両方同時にプライベートネットワークになります。

パブリックネットワークに対しては Windows のファイアーウォールがデフォルトで厳しめの制限をかけているため、
ゲストの仮想マシンとホストPCとの間の通信ができないことがあります。
その場合は上記の方法でプライベートネットワークとして扱うか、
あるいは次のポストで説明するようにファイアーウォールのルールを修正するか、いずれかの対応が必要となります。


[Windows10 Client Hyper-V にて CentOS7 box に NAT 環境を構築する (2/2)]({{< relref "howto_config_hyperv_nat_box.md" >}}) に続きます。
