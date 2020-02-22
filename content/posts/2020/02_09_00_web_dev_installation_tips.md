---
title: "echoの実行環境構築メモ"
description: "WSL上でgoやったりnode.jsやったり"
tags: ["golang", "node.js", "WSL"]
date: 2020-02-09T03:57:59+09:00
lastmod: 2020-02-16T23:15:00+09:00
archives: 2020/02/09
draft: false
---

## 追記 (2020/02/16 23:15)

フロント側のfirebase認証に必要な設定が足りなかったので追記。

---

現在、(完成するかわからないというかそれまで続くかわからないが)[完全匿名SNS](https://github.com/suihan74/echo)を作りたいなと思い勉強しながらちまちまやっている。

複数の環境で開発を行うにあたり「最初どうやって環境構築したっけ……」といったことが頻発したので三回目以降悩まないように手順をメモしておくとする。

## WSLのインストール

当方はWindows環境であるので、とりあえずWSLでも用意しておくのが楽でよい。

1. 「Windowsの機能の有効化または無効化」で`Windows Subsystems for Linux`を有効化する

2. Microsoft Storeを開いてUbuntuをインストールする  
   (ディストリビューションは今日日べつに何でもいいが)

3. [mintty/wsltty](https://github.com/mintty/wsltty)をインストールして見た目を適当にいい感じにしておく  
   (やらなくてもよい)

次にGoやNodeのインストールを行うのだが、WSLでaptを使って入れられるバージョンは古いので、まずそれぞれのバージョン管理用のツールをインストールしてそれを使って導入する。

## Goのインストール

### [goenv](https://github.com/syndbg/goenv)のインストール

```sh
$ git clone git@gihub.com:syndbg/goenv ~/.goenv
```

.bash_profileなりを編集。

```sh
export GOENV_ROOT=$HOME/.goenv
export PATH=$GOENV_ROOT/bin:$PATH
eval '$(goenv init -)'
```

反映。

```sh
$ source ~/.bash_profile
```

### 適当なバージョンのGoをインストール

```sh
# インストール可能なバージョン一覧を表示
$ goenv install -l
# インストール
$ goenv install 1.13.7
```

バージョンの確認。

```sh
$ go version
go version go1.13.7 linux/amd64
```

## Node.jsのインストール

### [nodebrew](https://github.com/hokaccha/nodebrew)のインストール

```sh
$ curl -L git.io/nodebrew | perl - setup
```

.bash_profileなりを編集。

```sh
export NODEBREW_ROOT=$HOME/.nodebrew/current
export PATH=$NODEBREW_ROOT/bin:$GOENV_ROOT/bin:$PATH
```

反映。

```sh
$ source ~/.bash_profile
```

### 適当なバージョンのNode.jsをインストール

```sh
# インストール可能なバージョン一覧を表示
$ nodebrew ls-remote
# インストール
$ nodebrew install v9.5.0
```

バージョンの確認。

```sh
$ node -v
v9.5.0
$ npm -v
v5.6.0
```

## PostgreSQLのインストールとプロジェクト用の設定

### インストール

```sh
$ sudo apt-get install postgresql
```

### プロジェクト用のユーザーを作成

```sh
$ sudo su postgres
$ createuser -d -P echo
```

ユーザー名`echo`、パスワード`echo`という適当っぷり。

### 認証設定の変更

デフォルトではUNIX側のユーザー名とPostgreSQL側のユーザー名が一致しない場合、正しいパスワードで`psql`コマンドでログインしようとしても失敗する(`peer`認証)

今回はそれだと困るので、とりあえず(DB側の)ユーザーechoに関しては`trust`なり`md5`なりに変更する。

```sh
$ sudo nano /etc/postgresql/10/main/pg_hba.conf
```

```conf {linenos=table, linenostart=90}
local   all             echo              md5
local   all             all              peer
```

他もすべて変えたいなら上の設定の`echo`の部分を`all`にすればいい。

### 参考

[PostgreSQLのPeer認証と他の認証方法への変更|Web Memorandum](http://www.utsushiiro.jp/blog/archives/327)

## firebase認証を使用するための設定

### フロント側のfirebaseのアプリ設定を修正する

`echo/front/src/main.js`内の`firebaseConfig`オブジェクトの内容を、  
「[firebaseコンソール](https://console.firebase.google.com/)/(プロジェクト)/設定/全般」最下部の`「マイアプリ」`部分の内容に修正する。

### バック側にfirebaseの秘密鍵を用意

「[firebaseコンソール](https://console.firebase.google.com/)/(プロジェクト)/設定/サービスアカウント」からAdminSDKの秘密鍵を生成して、`echo/back/echo-sns-firebase-adminsdk.json`として配置する。

Goのコードでは環境変数でファイルを指定して秘密鍵の内容を読み込むようにしているので、`CREDENTIALS`という名前で環境変数に先のファイル名を指定する。

.bash_profileにでも書いておく。

```sh
export CREDENTIALS=$HOME/echo/back/echo-sns-firebase-adminsdk.json
```

なんか気の利いた名前に変えておいたほうが良さそう。

## ローカルで実行

### 1. PostgreSQLのサービス起動

```sh
$ sudo service postgresql start
```

### 2. バックエンドの起動

```sh
$ cd echo/back
$ go run *.go
```

`go mod`のおかげで依存パッケージは初回実行時に自動的に用意される。

### 3. フロントエンドの起動

初回起動前に依存パッケージをインストールする。

```sh
$ cd echo/front
$ npm install
```

しばらくかかるので待つ。本当におまえそんなに色々必要か？？？って気持ちになる。

開発版での実行。

```sh
$ npm run dev
```
