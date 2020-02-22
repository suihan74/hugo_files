---
title: "GitHub Actionsでブログを自動デプロイするようにした"
description: "サブモジュールを使っているリポジトリ上でのHugo+Actionsの設定"
tags: ["Hugo", "GitHub", "Actions"]
date: 2020-01-29T02:40:00+09:00
lastmod: 2020-01-29T03:00:00+09:00
archives:
    - 2020
    - 2020-01
    - 2020-01-29
draft: false
---

## 追記 (2020/01/29 03:00)

Actionのジョブを実行する際のタイムゾーンを指定するようにymlファイルを修正。

```yml
env:
  TZ: 'Asia/Tokyo'
```

---

## 前提

このブログは次のような状態で運用している（していた）。  
「Hugoのデータを置いているリポジトリ」に「公開用のhtmlファイルが出力されるディレクトリ」や「テーマファイルが配置されたディレクトリ」がサブモジュールとして配置されている。

1. `suihan74/hugo_files`にMarkdownで記事を書く

    ```bash
    hugo new posts/2020/hogehoge.md
    ```

2. Hugoで`public/`以下に静的ページ生成をしてpushする

    - `public/`はリポジトリ`suihan74/suihan74.github.io`にpushする

    - `public/`と、テーマファイルを配置している`themes/`はそれぞれ`hugo_files`のサブモジュールとして配置している

3. <https://suihan74.github.io/>に変更が反映される

今までは億劫がってCI的なことしていなかったので、記事を書くたびに「`public/`を`suihan74.github.io`にpushして」「`hugo_files`をpushする」みたいな手間をかけていた。

[![suihan74/hugo_files - GitHub](https://gh-card.dev/repos/suihan74/hugo_files.svg?fullname=)](https://github.com/suihan74/hugo_files)

[![suihan74/suihan74.github.io - GitHub](https://gh-card.dev/repos/suihan74/suihan74.github.io.svg?fullname=)](https://github.com/suihan74/suihan74.github.io)

## やったこと

[GitHub Actions](https://github.co.jp/features/actions)を利用して、`hugo_files`に記事をpushしたらGitHub側で勝手に`public/`を生成して`suihan74.github.io`にデプロイするようにした。

色々よしなにやってくれるActionが公開されているので使わせてもらう。神。

[![actions/checkout - GitHub](https://gh-card.dev/repos/actions/checkout.svg)](https://github.com/actions/checkout)

[![peaceiris/actions-gh-pages - GitHub](https://gh-card.dev/repos/peaceiris/actions-gh-pages.svg)](https://github.com/peaceiris/actions-gh-pages)

[![peaceiris/actions-hugo - GitHub](https://gh-card.dev/repos/peaceiris/actions-hugo.svg)](https://github.com/peaceiris/actions-hugo)

## 方法

### 1. 認証に必要なものを準備する

**sshキーを使う方法**と**個人アクセストークンを使う方法**、好きな方を選ぶ。なんか検索してよく出てくるのは前者。

#### sshキーを使う方法

1. ローカル環境でssh-keygenを使って**公開鍵**と**秘密鍵**を生成する

    ```bash
    ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
    ```

2. **秘密鍵**を`hugo_files`リポジトリのSecretsに追加する

    `Settings > Secrets`にNameを「ACTIONS_DEPLOY_KEY」として、Valueには**秘密鍵**の中身をコピペする。
    Nameは後で記述する設定ファイルと整合していれば別に何でもいい。

3. **公開鍵**を`suihan74.github.io`リポジトリの`Deploy keys`に追加する

    `Settings > Deploy keys`に**公開鍵**の中身をコピペする。名前は別に何でもいい。

---

#### 個人アクセストークンを使う方法

1. アクセストークンを生成する

    <https://github.com/settings/tokens>にアクセスして以下の内容に従ってアクセストークンを生成する。

    > [コマンドライン用の個人アクセストークンを作成する - GitHub ヘルプ](https://help.github.com/ja/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line)

2. アクセストークンを`hugo_files`リポジトリのSecretsに追加する

    `Settings > Secrets`にNameを「MY_GITHUB_ACCESS_TOKEN」として 1. で作成したアクセストークンを設定する。  
    Nameは後で記述する設定ファイルと整合していれば別に何でもいい。

### 2. Actions用のディレクトリと設定ファイルを用意する

`hugo_files/.github/workflows/main.yml`を作成する。GitHubのリポジトリページ上のメニューから作成するなりローカルで作ってpushするなりうまくやる。

ファイル名は別に何でもいい。

### 3. 設定ファイルを記述する

#### .github/workflows/main.yml

```yml
name: GitHub Pages

on:
  push:
    branches:
    - master

jobs:
  build-deploy:
    runs-on: ubuntu-18.04
    steps:
    - name: Fix up git URLs
      run: echo -e '[url "https://github.com/"]\n  insteadOf = "git@github.com:"' >> ~/.gitconfig

    - name: Checkout
      uses: actions/checkout@v1
      with:
        submodules: true

    - name: Setup Hugo
      uses: peaceiris/actions-hugo@v2

    - name: Build
      run: hugo --gc --minify --cleanDestinationDir
      env:
        TZ: 'Asia/Tokyo'

    - name: Deploy
      uses: peaceiris/actions-gh-pages@v2
      env:
        ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
       # 個人アクセストークンを使う場合
       #PERSONAL_TOKEN: ${{ secrets.MY_GITHUB_ACCESS_TOKEN }}
        EXTERNAL_REPOSITORY: suihan74/suihan74.github.io
        PUBLISH_BRANCH: master
        PUBLISH_DIR: ./public
        TZ: 'Asia/Tokyo'
```

## 結果

これで今後、記事の更新に関しては`hugo_files`にpushするだけで自動的に更新が反映されるようになった。

やったぜ。

## 参考

- [GitHub Actions による GitHub Pages への自動デプロイ - Qiita](https://qiita.com/peaceiris/items/d401f2e5724fdcb0759d)

- [Hugo+GitHub Pages+GitHub Actionsで自動デプロイする方法 - Qiita](https://qiita.com/syui/items/f1f2db8660cda34d8938)

- [HugoのデプロイをGitHub Actionsで行う &middot; Posts of knsh14](https://knsh14.github.io/posts/how-to-automate-deploying-hugo/)
