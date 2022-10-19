---
title: "CS:GOサーバーの作り方"
date: 2022-10-19T18:43:17+09:00
draft: false
---

Counter-Strike: Global Offensive(以下、CS:GO)では、ユーザーがサーバーを構築しプレイヤー同士で遊ぶことができます。  
コミュニティ大会だけでなくメジャー大会でも同じような構成でサーバーを運用しています。

CS:GOサーバーはMinecraft ForgeのようなMod(プラグイン)を導入し、標準にはない機能を導入したり、試合を管理することができます。  
今回は、CS:GOサーバーのインストールからプラグインを動かすために必要なMetamodとSourceModのインストールまで紹介します。

## 説明の前に

OSはLinuxを前提に手順を紹介します。Windowsでも構築は可能ですが、ここでは紹介しません。  
また、各OSを以下の名称で紹介します。それ以外は各OSに合わせてご参考ください。  

- Debian, Ubuntu → Debian系
- CentOS, CentOS Stream, Rocky Linux, AlmaLinux → RHEL系

## 必要なもの

- サーバー (最低でも 1コア/2GB)
- 安定した回線

動かすためのサーバーはVPSやAWSのようなクラウドサービス、自宅のサーバーを用意してください。  
**最低でも1コア/2GBと安定した回線**があれば安定すると思いますが、提供元のサービスによってはスケールアップしても安定しないときがあります。  

## OS周りの準備

### 必要パッケージのインストール

セットアップに必要なOSパッケージをインストールします。  
各OSによってコマンドやパッケージが異なるので合わせて使用してください。

#### Debian系

##### パッケージの更新

```sh {linenos=false}
apt update && apt upgrade -y
```

##### 必要パッケージのインストール

```sh {linenos=false}
apt install -y curl tar vim lib32gcc-s1
```

#### RHEL系

##### パッケージの更新

```sh {linenos=false}
dnf update -y
```

##### 必要パッケージのインストール

```sh {linenos=false}
dnf install -y curl tar vim glibc.i686 libstdc++.i686
```

### ユーザーの作成

rootユーザーで運用するのはセキュリティの理由から良くないのでユーザーを作成します。

#### Debian系

```sh {linenos=false}
adduser steam
```

名前などの入力を求められますが全て空でOKです。

#### RHEL系

```sh {linenos=false}
useradd steam
```

### sudo権限の許可

作成したユーザーがsudoコマンドを使えるように各グループへ参加させます

#### Debian系

```sh {linenos=false}
usermod -aG sudo steam
```

#### RHEL系

```sh {linenos=false}
usermod -aG wheel steam
```

## SteamCMDのインストール

### steamユーザーに切り替え

```sh {linenos=false}
su - steam
```

### SteamCMDのインストール

```sh {linenos=false}
mkdir ~/steamcmd && curl -sL "https://media.steampowered.com/installer/steamcmd_linux.tar.gz" | tar -zxv -C ~/steamcmd
```

## CS:GOサーバーのインストール

SteamCMDを使ってCS:GOサーバーをインストールします。  
オプションを渡さずに実行すれば対話式で操作することもできます。

```sh {linenos=false}
./steamcmd/steamcmd.sh +force_install_dir /home/steam/srcds +login anonymous +app_update 740 validate +quit
```

## CS:GOサーバー設定

ここでCS:GOサーバーをインストールしたディレクトリへ移動しておきます。

```sh {linenos=false}
cd ~/srcds
```

### 起動ファイルのインストール

```sh {linenos=false}
curl -sL -o start.sh "https://raw.githubusercontent.com/k4tyxd/csgo-server-startup/main/start.sh"
```

### 自動アップデートの手順ファイルをインストール

CS:GOサーバーが立ち上がる前にインストールしたSteamCMDを使って自動アップデートしてくれます。  
そのための手順が記載されたファイルを設置します。

```sh {linenos=false}
curl -sL -o autoupdate.txt "https://raw.githubusercontent.com/k4tyxd/csgo-server-startup/main/autoupdate.txt"
```

### サーバー設定ファイルのインストール

```sh {linenos=false}
curl -sL -o csgo/cfg/server.cfg "https://raw.githubusercontent.com/k4tyxd/csgo-server-startup/main/csgo/cfg/server.cfg"
```

インストールしたあとエディタを使って編集します。vimエディタを初めて使う場合は調べてきてください。

```sh {linenos=false}
vim csgo/cfg/server.cfg
```

- `hostname`: サーバーの名前を指定します。コミュニティサーバーにも表示されます。
- `sv_password`: サーバー接続の際に使用するパスワードです。
- `sv_tags`: コミュニティサーバーで検索する際のキーワードです。複数ある場合はコンマで区切ります。
- `rcon_password`: リモートコントロール(rcon)用のパスワードです。必ず変更してください。
- `sv_setsteamaccount`: [こちら](https://steamcommunity.com/dev/managegameservers)から取得したトークンを入力します。

### Metamodのインストール

特定のバージョンをインストールしたい場合は以下のリンクから使用してください。
https://www.sourcemm.net/downloads.php?branch=stable

```sh {linenos=false}
curl -sL "https://www.sourcemm.net/latest.php?version=1.11&os=linux" | tar -zxv -C csgo
```

CS:GOで動くように`metamod.vdf`の内容を編集します。

```sh {linenos=false}
sed -i -e 's|addons|../csgo/addons|g' csgo/addons/metamod.vdf csgo/addons/metamod_x64.vdf
```

### SourceModのインストール

特定のバージョンをインストールしたい場合は以下のリンクから使用してください。
https://www.sourcemod.net/downloads.php?branch=stable

```sh {linenos=false}
curl -sL "https://www.sourcemod.net/latest.php?version=1.11&os=linux" | tar -zxv -C csgo
```

## ファイル権限

CS:GOサーバーのファイル権限をsteamユーザーが扱えるようにします。

```sh {linenos=false}
sudo chmod -R 700 ~/srcds && sudo chown -R steam. ~/srcds
```

## 起動

```sh {linenos=false}
./start.sh
```

このままではSSHのセッションが切れるとCS:GOサーバーのプロセスも終了してしまうため、screenやtmuxを使ってセッションを維持するのをおすすめします。
